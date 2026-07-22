# IP Address Versions

The **IP version** is the numeric identifier that tells network devices which format an [IP-Address](IP-Address.md) uses and how to parse the packet header. Although the version field can hold several values, only **IPv4** and **IPv6** are in general production use today — there is no deployed **IPv1**, **IPv2**, **IPv3**, or **IPv5** standard.

## Overview

Every IP packet begins with a 4-bit **Version** field (the first 4 bits of the IP header), so an interface can carry addresses of different versions and a receiver knows immediately how to interpret what follows. Over the history of TCP/IP, version numbers 0 through 9 were assigned to various experimental and production protocols, but the two that matter operationally are **IPv4** (32-bit addressing) and **IPv6** (128-bit addressing).

Understanding the versions is the foundation for everything else in [Networking-Fundamentals](Networking-Fundamentals.md): how addresses are partitioned by a [Network-Mask-Subnet-Mask-Net-Mask](Network-Mask-Subnet-Mask-Net-Mask.md), how [addresses are validly assigned](Rules-for-Assigning-an-IP-Address-to-a-Device.md), and where addressing sits in [the OSI / TCP-IP models](The-OSI-Model-and-TCP-IP-Model.md) (the Network / Internet layer).

> [!NOTE]
> **The version lives in the header**
> The version number is not part of the address text — it is a 4-bit field at the very start of the IP header. That is why a `192.168.1.10` (IPv4) and a `2001:db8::1` (IPv6) packet can traverse the same wire and be dispatched correctly.

## Internet Protocol Version History

| Version | Status | Description |
| --- | --- | --- |
| **IPv1** | Experimental | Early version used during the development of TCP/IP; never publicly deployed. |
| **IPv2** | Experimental | Internal research version; never standardized for Internet use. |
| **IPv3** | Experimental | Known as ST (Stream Protocol), designed for real-time traffic; not adopted as a replacement for IP. |
| **IPv4** | Active | Current widely used protocol using 32-bit addresses (e.g., `192.168.1.10`). |
| **IPv5** | Experimental | Version number assigned to the Internet Stream Protocol (ST-II); never intended to replace IPv4. |
| **IPv6** | Active | Successor to IPv4 using 128-bit addresses (e.g., `2001:db8::1`). |

## Why Is There No IPv5?

The version number **5** was already assigned to the **Internet Stream Protocol (ST / ST-II)**, an experimental protocol developed for voice and video streaming. Because that version number was already in use, the next Internet Protocol designed to succeed IPv4 was assigned version **6**, resulting in **IPv6**.

## IPv4 vs IPv6

The two production versions differ far beyond address length. The move to IPv6 was driven mainly by **IPv4 address exhaustion** — a 32-bit space allows roughly 4.3 billion addresses, which the modern Internet outgrew.

| Feature | IPv4 | IPv6 |
| --- | --- | --- |
| Address size | 32-bit | 128-bit |
| Notation | Dotted decimal (`192.168.1.10`) | Hexadecimal, colon-separated (`2001:db8::1`) |
| Approx. address count | ~4.3 billion | ~3.4 x 10^38 |
| Header | Variable length, includes checksum | Fixed 40-byte base header, no checksum |
| Address configuration | Manual or DHCP | SLAAC (autoconfig) or DHCPv6 |
| Broadcast | Yes | No — replaced by multicast / anycast |
| Fragmentation | Router or host | Host only |

> [!TIP]
> **Both stacks usually run at once**
> Most modern hosts, including Windows, run **dual-stack** — IPv4 and IPv6 are active simultaneously. Do not assume a target or network is "IPv4 only" just because that is how it is documented.

## Common IP Address Formats

### IPv4 (32-bit)

```text
192.168.1.1
10.0.0.1
172.16.1.100
```

### IPv6 (128-bit)

```text
2001:db8::1
fe80::1
2606:4700:4700::1111
```

### Checking the address versions on a host

On Windows, list all configured IPv4 and IPv6 addresses:

```cmd
ipconfig /all
```

On Linux, show IPv4 and IPv6 addresses separately:

```bash
ip -4 addr
ip -6 addr
```

## Security Considerations

> [!WARNING]
> **IPv6 is the forgotten attack surface**
> On dual-stack networks, IPv6 is frequently enabled but **unmonitored and unfiltered**. Defensive controls (firewall rules, IDS/IPS signatures, ACLs) are often written for IPv4 only, so IPv6 can provide an unwatched path for lateral movement, exfiltration, and rogue services. `fe80::/10` link-local addresses in particular are auto-configured and reachable on the local segment even where IPv6 was never intentionally deployed.

- **Scan both stacks.** An IPv4-only scan misses IPv6-reachable services. When enumerating a target, account for IPv6 addresses discovered via neighbor discovery, DNS `AAAA` records, or link-local traffic — see Network-Reconnaissance-Scanning.
- **Rogue Router Advertisements (RA).** Because IPv6 hosts auto-configure from RAs (SLAAC), a spoofed RA can redirect traffic — the IPv6 analog of rogue-DHCP / MITM attacks. Enable **RA Guard** on switches.
- **Larger address space cuts both ways.** IPv6 subnets are too large to brute-force scan host-by-host, but hosts are still discoverable through DNS, multicast, and neighbor caches.
- **Address text is not identity.** As with a [Media-Access-Control(MAC)-Address](Media-Access-Control(MAC)-Address.md), an IP address (either version) can be spoofed and must never be treated as an authentication boundary on its own.

## Best Practices

- If IPv6 is not in use, disable it deliberately and consistently — do not leave it half-enabled and unmonitored.
- Apply firewall, IDS/IPS, and ACL policy to **both** IPv4 and IPv6 with equivalent rigor.
- Enable switch protections (RA Guard, DHCPv6 Guard) to block rogue IPv6 configuration.
- Document which subnets and services are dual-stack so no addressing path goes untracked.
- Standardize on **IPv4 + IPv6** going forward; treat the experimental legacy versions as historical trivia only.

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Host reachable by IPv4 but not IPv6 (or vice versa) | Only one stack is configured/routed — verify both with `ipconfig /all` (Windows) or `ip addr` (Linux) and check the matching default gateway/route |
| Unexpected `fe80::` traffic on a segment | Normal IPv6 link-local auto-configuration; confirm it is expected and that IPv6 is intended on that network |
| Scanner reports no IPv6 hosts on a busy network | Scan is IPv4-only — explicitly target IPv6 ranges / `AAAA` records |

## References

- [RFC 791 — Internet Protocol (IPv4)](https://www.rfc-editor.org/rfc/rfc791)
- [RFC 8200 — Internet Protocol, Version 6 (IPv6) Specification](https://www.rfc-editor.org/rfc/rfc8200)
- [RFC 1190 — Experimental Internet Stream Protocol, Version 2 (ST-II)](https://www.rfc-editor.org/rfc/rfc1190)
- [IANA — IP Version Numbers registry](https://www.iana.org/assignments/version-numbers/version-numbers.xhtml)

## Related

- [IP-Address](IP-Address.md) — the addressing concept these versions (IPv4/IPv6) implement
- [Network-Mask-Subnet-Mask-Net-Mask](Network-Mask-Subnet-Mask-Net-Mask.md) — how address versions are partitioned into network/host portions
- [Rules-for-Assigning-an-IP-Address-to-a-Device](Rules-for-Assigning-an-IP-Address-to-a-Device.md) — assignment rules that depend on the IP version
- [The-OSI-Model-and-TCP-IP-Model](The-OSI-Model-and-TCP-IP-Model.md) — where IP addressing sits in the network stack
- [Networking-Fundamentals](Networking-Fundamentals.md) — module overview
- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
