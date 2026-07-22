# VirtualBox Network Modes

When building lab environments in VirtualBox, the network mode chosen for each adapter determines how a virtual machine (VM) communicates with the host, with other VMs, and with external networks. Picking the correct mode is what separates a safe, isolated attacker/victim lab from a VM that is accidentally exposed on the corporate LAN.

## Overview

Each VM can have up to four network adapters, and each adapter is independently set to one of the modes below. A common lab pattern is two adapters: one **NAT** for internet/updates plus one **Host-Only** or **Internal** for the isolated attack network.

## Concepts

### Network Address Translation (NAT)

- **Best for:** Internet access only (simple labs).
- VM can access the internet.
- VM cannot be accessed directly from the host or other machines.
- VirtualBox performs NAT behind the host.
- VM typically receives an IP like `10.0.2.x`.

**Use case**

- Installing packages (`apt update`, `pip install`, etc.)
- Web browsing from Kali Linux
- Safe, isolated internet access

**Limitation**

- No inbound connections unless port forwarding is configured.

### Bridged Adapter

- **Best for:** Making the VM behave like a real machine on your LAN.
- VM gets an IP from your router (e.g., `192.168.1.x`).
- Appears as a separate device on the local network.
- Can communicate with the host system, other LAN machines, and the internet.

**Use case**

- Realistic network simulation
- Testing against real LAN devices
- Hosting services accessible from the network

> [!WARNING]
> **Bridged mode exposes the VM on the physical network**
> The VM is visible to every device on the LAN. Do not use Bridged mode in a corporate environment without authorization — a deliberately-vulnerable target on the real network is a serious risk.

### Internal Network

- **Best for:** Fully isolated lab environments.
- Only VMs using the same internal network name can communicate.
- No internet access.
- Host cannot access the VMs.

**Use case**

- Attacker + victim lab setup
- Malware analysis sandbox
- CTF-style isolated environments

**Example**

- Kali Linux VM
- Metasploitable VM
- Both connected to `intnet-lab`

### Host-Only Adapter

- **Best for:** Communication between host and VM.
- VM can communicate with the host machine and other Host-Only VMs.
- No internet access (unless a second adapter is added).
- VirtualBox typically assigns IPs in the range `192.168.56.x`.

**Use case**

- Local pentesting lab
- Testing services from the host browser
- Safer alternative to Bridged mode

### NAT Network

- **Best for:** Multiple VMs with internet access.
- Similar to NAT, but multiple VMs share the same NAT network.
- VMs can communicate with each other.
- Internet access is available.

Configured via:

```text
File → Tools → Network Manager
```

**Use case**

- Multi-machine lab with internet access
- Kali attacking Metasploitable while retaining internet connectivity

## Examples

Quick comparison of the five modes:

| Mode | Internet | VM ↔ Host | VM ↔ VM | Visible on LAN |
|---|---|---|---|---|
| NAT | Yes | Only via port forwarding | No | No |
| Bridged | Yes | Yes | Yes | Yes |
| Internal | No | No | Yes | No |
| Host-Only | No | Yes | Yes | No |
| NAT Network | Yes | Only via port forwarding | Yes | No |

## GUI Steps

> [!NOTE]
> **Screenshot**
> ![VirtualBox VM Settings → Network tab showing Adapter 1 attached to "Host-Only Adapter" with the vboxnet0 network selected](placeholder-screenshot)

> [!NOTE]
> **Screenshot**
> ![VirtualBox Network Manager (File → Tools → Network Manager) showing a defined NAT Network named "labnet" with its 10.0.x.0/24 range](placeholder-screenshot)

## Security Considerations

- For any deliberately-vulnerable target, prefer **Internal** or **Host-Only** so the VM cannot reach the internet or the physical LAN.
- Use **NAT** or a dedicated **NAT Network** only when the guest genuinely needs internet (updates, tooling) and combine it with a snapshot you can roll back to.
- Never place an unpatched lab machine on **Bridged** mode inside a production/corporate network.

## Best Practices

- Two adapters per lab VM: NAT for updates + Internal/Host-Only for the isolated attack path.
- Give the isolated network a descriptive name (`intnet-lab`, `labnet`) and reuse it across every VM that must talk to each other.
- Disable the internet-facing adapter before running untrusted code.

## References

- VirtualBox Manual — Virtual Networking: <https://www.virtualbox.org/manual/ch06.html>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Virtual-Networking](Virtual-Networking.md) — network modes across all hypervisors — related note
- [Network-Address-Translation(NAT)](../Proxy-Server-Administration/Network-Address-Translation(NAT).md) — NAT is a core VM network mode — related note
- [Port-Forwarding](../Proxy-Server-Administration/Port-Forwarding.md) — forwarding ports into NAT VMs — related note
- [Lab-Design](Lab-Design.md) — where these modes fit in the full lab topology — related note
