# Exclusion Range in DHCP

An **Exclusion Range** is a set of IP addresses within a DHCP scope that the DHCP server is instructed **not to assign** to clients. These IPs are held back for **manual/static assignment** or for **devices that require fixed addresses**, such as printers, servers, gateways, or other network hardware.

## Overview

A [scope](Scope-in-a-DHCP-Server.md) defines the full pool of addresses a DHCP server can lease for a subnet. In practice, some of those addresses must never be handed out dynamically because they belong to infrastructure that is configured statically — routers, DNS servers, out-of-band management interfaces, and the like. An exclusion range carves those addresses out of the leasable pool while keeping them documented inside the scope definition, so the administrator retains a single, authoritative view of the subnet.

Exclusions are a common companion to [DHCP-Reservations](DHCP-Reservations.md) and [DHCP-Scope-Options](DHCP-Scope-Options.md): an exclusion keeps addresses *out* of dynamic assignment entirely, whereas a reservation keeps a specific address bound to one device while still managing it through DHCP.

> [!NOTE]
> **Exclusion is a subtractive rule**
> The scope still *contains* the excluded addresses on paper — the DHCP server simply skips them when choosing an address to offer. This is why exclusions are the cleanest way to reconcile a statically-addressed infrastructure block with an otherwise DHCP-managed subnet.

## How It Works

When a scope is created, the server treats every address between the scope's start and end as a candidate for lease. Adding an exclusion range tells the server to remove that block from the set of offerable addresses. During the [DORA handshake](DORA-Process.md), the server chooses an available address from the remaining pool — excluded addresses are never included in a DHCP **Offer**.

Because the exclusion is enforced server-side, a device that has been given a static address inside the excluded block will not collide with a lease, and no client will ever be offered that address.

## Example Scenario

- Suppose a DHCP scope is defined as:

```text
192.168.1.1 - 192.168.1.254
```

- But you want to **exclude** the following range:

```text
192.168.1.1 - 192.168.1.20
```

- In this case, IPs `192.168.1.1` through `192.168.1.20` will never be assigned dynamically by the DHCP server. They can be used for:

	- Routers/gateways

	- DNS servers

	- Static IP devices

## Why Use Exclusion Ranges?

- To **prevent conflicts** between static and dynamic addresses.

- To **preserve IPs** for infrastructure devices.

- To **simplify network management** by keeping known devices at known addresses.

## Exclusion vs. Reservation

| Feature | Exclusion Range | Reservation |
|---|---|---|
| Purpose | Keep IPs **out** of the dynamic pool | Always hand the **same IP** to one MAC |
| Bound to a device? | No | Yes (MAC / client-ID) |
| Typical use | Gateways, static servers, out-of-band mgmt | Printers, VoIP phones, servers you still want DHCP-managed |

> [!TIP]
> **Exclude, then reserve**
> If a device must always receive the *same* address but you still want DHCP to push option changes (DNS, gateway, NTP) to it centrally, use a [reservation](DHCP-Reservations.md) rather than an exclusion plus a static config. Reserve inside the pool; exclude only what is truly hand-configured.

## Configuration

### Windows Server — `netsh`

```cmd
netsh dhcp server scope 192.168.1.0 add excluderange 192.168.1.1 192.168.1.20
```

### Windows Server — PowerShell

```powershell
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 -StartRange 192.168.1.1 -EndRange 192.168.1.20
Get-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0   # verify
```

### Linux — `isc-dhcp-server`

ISC DHCP has no explicit "exclude" keyword; you simply narrow the `range` so the excluded block is never offered:

```bash
subnet 192.168.1.0 netmask 255.255.255.0 {
  # .1–.20 kept for static/infra devices by starting the pool at .21
  range 192.168.1.21 192.168.1.254;
}
```

## Security Considerations

> [!WARNING]
> **Exclusions are for hygiene, not defense**
> An exclusion range prevents address *conflicts* — it does not authenticate anything. DHCP itself is unauthenticated, so exclusions do nothing to stop a [rogue DHCP server](Rogue-DHCP-Server.md) or a [starvation attack](DHCP-Starvation-Attack.md). Do not treat "the infrastructure block is excluded" as a security control.

From an offensive perspective, the layout of a scope is reconnaissance value: the excluded low range (e.g. `.1`–`.20`) is a strong hint about where **gateways, DNS, and management interfaces** live, since administrators conventionally park static infrastructure at the bottom of the subnet. An attacker who can read scope configuration (via a compromised server or `Get-DhcpServerv4ExclusionRange`) effectively gets a map of high-value targets.

Defensively, exclusions belong alongside the real controls: authorize DHCP servers in Active Directory, enable [DHCP-Snooping](DHCP-Snooping.md) on access switches, and pair it with Dynamic ARP Inspection and IP Source Guard to close man-in-the-middle paths. See [DHCP-Security-Issues-and-Attacks](DHCP-Security-Issues-and-Attacks.md) for the full attack surface.

## Best Practices

- Reserve the bottom (or top) of each scope as an excluded infrastructure block and document what lives there in one place.
- Prefer **reservations** over exclusion-plus-static when you still want DHCP to manage a device's options centrally.
- Keep exclusion ranges consistent across subnets (e.g. always `.1`–`.20`) so the addressing plan is predictable to operators — while being aware this predictability also aids attackers.
- Re-verify exclusions after resizing a scope; growing a scope can re-expose addresses you intended to keep out.
- Never rely on exclusions as a security boundary — layer snooping and server authorization on top.

## Troubleshooting

| Symptom | Likely cause & fix |
|---|---|
| A static device intermittently loses connectivity / duplicate-IP warnings | Its address was not actually excluded and got leased to a client — add the address to an exclusion range (or reserve it). |
| Clients fail to get a lease after adding an exclusion | The exclusion overlaps too much of the scope, exhausting the usable pool — shrink the exclusion or enlarge the scope. |
| Exclusion appears ignored | Exclusion applied to the wrong `ScopeId`, or the scope is inactive — verify with `Get-DhcpServerv4ExclusionRange -ScopeId <scope>`. |

## References

- [DHCP scopes and exclusions (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/networking/technologies/dhcp/dhcp-top)
- [Add-DhcpServerv4ExclusionRange (Microsoft Learn)](https://learn.microsoft.com/en-us/powershell/module/dhcpserver/add-dhcpserverv4exclusionrange)
- [RFC 2131 — Dynamic Host Configuration Protocol](https://www.rfc-editor.org/rfc/rfc2131)

## Related

- [Scope-in-a-DHCP-Server](Scope-in-a-DHCP-Server.md) — scope the exclusion applies to
- [DHCP-Reservations](DHCP-Reservations.md) — complementary static-assignment feature
- [DHCP-Scope-Options](DHCP-Scope-Options.md) — scope configuration context
- [DORA-Process](DORA-Process.md) — lease handshake that skips excluded addresses
- [DHCP-Security-Issues-and-Attacks](DHCP-Security-Issues-and-Attacks.md) — why exclusions are not a security control
- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub
