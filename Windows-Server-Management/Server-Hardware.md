# Server Hardware

Server hardware is the class of physical computing equipment built for continuous, high-load operation — engineered for reliability, redundancy, scalability, and remote manageability rather than the single-user experience of a desktop. It is the foundation every Windows Server role in this course ultimately runs on.

## Overview

Where a consumer PC optimizes for cost and single-user responsiveness, a server optimizes for **uptime and throughput under sustained multi-user load**. That difference shows up in almost every component: error-correcting memory, redundant and hot-swappable power supplies, multi-socket CPUs, RAID-backed storage, and out-of-band management processors that let an administrator control the box even when the operating system is down. Understanding this hardware matters both for administration — sizing a host for a role such as [AD DS](../Active-Directory-Domain-Services-AD-DS/Active-Directory-Domain-Services.md) — and for offensive work, because the out-of-band management plane is a network-reachable target in its own right.

See [CPU-Architecture](../Fundamental-Of-Operating-System/CPU-Architecture.md) for processor internals and [Laptop-or-PC-Specifications](../Fundamental-Of-Operating-System/Laptop-or-PC-Specifications.md) for the consumer-hardware contrast.

## Form Factors

- **Rack Servers** — fit into standardized 19-inch racks, sized in rack units (**1U, 2U, 4U**). The workhorse of most data centers.
- **Blade Servers** — slim, modular servers that slot into a shared **blade chassis**, pooling power, cooling, and networking across many blades for density.
- **Tower Servers** — standalone, desktop-like chassis but built for heavier duty; common in small offices and branch sites.
- **Mainframes & Enterprise Servers** — very high-performance systems specialized for massive, mission-critical workloads.

## Core Components

- **CPU (Processors)**
    - Multi-core, high-performance CPUs (Intel **Xeon**, AMD **EPYC**).
    - Designed for parallel processing, virtualization, and scalability; server boards commonly support **multiple CPU sockets**.
- **Memory (RAM)**
    - **ECC (Error-Correcting Code)** RAM detects and corrects single-bit memory errors for reliability.
    - Large capacity — from hundreds of GBs to several TBs in enterprise servers.
- **Storage**
    - **HDDs** — high-capacity, lower cost, slower.
    - **SSDs / NVMe** — high-speed, low-latency access.
    - Often configured in **RAID** for redundancy and/or performance.
- **Motherboard**
    - Server-grade boards with multiple CPU sockets and expansion slots.
    - Supports hot-swappable components.
- **Power Supply Units (PSU)**
    - **Redundant, hot-swappable** PSUs so a single supply failure does not cause downtime.
- **Cooling Systems**
    - High-efficiency fans; liquid cooling in high-density racks.
- **Network Interface Cards (NICs)**
    - Multiple Gigabit or **10 / 25 / 40 / 100 Gbps** interfaces.
    - Redundant connections (teaming/bonding) for failover.
- **RAID Controllers / Storage Adapters**
    - Hardware controllers that manage disks across RAID levels.

### RAID Levels at a Glance

| Level | Technique | Trade-off |
|-------|-----------|-----------|
| **RAID 0** | Striping, no redundancy | Fastest, but any disk loss = total loss |
| **RAID 1** | Mirroring | Full redundancy, 50% usable capacity |
| **RAID 5** | Striping + distributed parity | Survives 1 disk loss, good read speed |
| **RAID 6** | Striping + double parity | Survives 2 disk losses |
| **RAID 10** | Mirror + stripe | High performance and redundancy, 50% usable |

> [!NOTE]
> **Hardware vs. software RAID**
> A dedicated **hardware RAID controller** offloads parity calculation and often adds a battery-backed write cache. On Windows, **Storage Spaces** provides a software alternative that pools disks without a controller — useful in the lab, but hardware RAID is still the norm for production databases and Domain Controllers.

## Additional Features

- **Remote Management (Out-of-Band)** — dedicated baseboard management controllers let administrators power-cycle, reinstall, and view the console independently of the OS:
    - **IPMI** — the vendor-neutral standard.
    - **iLO** (HPE), **iDRAC / DRAC** (Dell), and equivalents from other vendors.
- **Hot-Swappability** — replace drives, PSUs, and fans without shutting the server down.
- **Virtualization Support** — hardware-assisted virtualization (**Intel VT-x**, **AMD-V**) underpins hypervisors and, on Windows, features like Hyper-V and virtualization-based security.
- **Scalability** — add CPUs, RAM, or storage without replacing the whole server.

## Server Hardware Vendors

| Vendor | Server Line |
|--------|-------------|
| Dell EMC | PowerEdge |
| HP Enterprise (HPE) | ProLiant |
| Lenovo | ThinkSystem |
| Cisco | UCS |
| IBM | Power Systems, Mainframes |
| Supermicro | SuperServer |

## Security Considerations

> [!WARNING]
> **The management plane is a hidden attack surface**
> Out-of-band controllers (**IPMI / iLO / iDRAC**) run a full embedded OS with network access that is **independent of the host OS and its patching**. Historically they have shipped with default credentials, exposed web interfaces, and serious vulnerabilities. A compromised BMC gives an attacker console access, the ability to mount virtual media, and effectively physical-equivalent control of the server — while remaining invisible to host-based EDR.

- **Isolate the management network** — IPMI/iLO/iDRAC interfaces belong on a dedicated, firewalled management VLAN, never on the general LAN or the internet.
- **Change default BMC credentials** and keep management firmware patched alongside the OS.
- **Firmware / supply-chain risk** — malicious or vulnerable BIOS/UEFI and BMC firmware can persist below the OS, surviving reinstalls. Hardware root-of-trust features (**TPM**, **Secure Boot**, silicon-level firmware verification) mitigate this.
- **Physical access = full compromise** — hot-swap bays and console ports mean an attacker with rack access can pull a disk or boot foreign media; pair hardware controls with disk encryption (e.g. BitLocker backed by a TPM).

## Best Practices

- Deploy **redundant PSUs, NICs, and RAID-protected storage** for any production role so no single component failure causes an outage.
- Use **ECC RAM** on every server — silent memory corruption is unacceptable for directory and database workloads.
- Put out-of-band management on a **segmented management network** with unique credentials and current firmware.
- **Size for the role**: match CPU cores, RAM, and disk I/O to the workload (a Domain Controller, a Hyper-V host, and a file server have very different profiles).
- Keep **BIOS/UEFI, BMC, and RAID firmware** in the patch cycle, not just the operating system.

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Server won't POST after a RAM upgrade | Non-ECC or unsupported/mismatched DIMMs, or wrong slot population — use vendor-qualified ECC modules and follow the board's population order |
| Degraded RAID array / predictive drive failure | Failing member disk — replace the hot-swap drive and let the controller rebuild; verify a spare and backups first |
| Can't reach the OS but need console access | Use the out-of-band controller (iLO/iDRAC/IPMI) for remote console and power control instead of a site visit |
| Random reboots or thermal shutdowns | Failed fan, blocked airflow, or a dead PSU in a redundant pair — check BMC hardware/event logs |

## References

- [Dell PowerEdge servers (documentation)](https://www.dell.com/support/home/en-us/products/server_int)
- [HPE ProLiant servers and iLO](https://www.hpe.com/us/en/servers/proliant-servers.html)
- [Intel Virtualization Technology (VT-x) overview](https://www.intel.com/content/www/us/en/virtualization/virtualization-technology/intel-virtualization-technology.html)
- [Trusted Platform Module (TPM) overview — Microsoft Learn](https://learn.microsoft.com/en-us/windows/security/hardware-security/tpm/trusted-platform-module-overview)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub
- [Windows-Server](Windows-Server.md) — OS that runs on this hardware
- [Windows-Server-Editions](Windows-Server-Editions.md) — editions and licensing that map to hardware tiers
- [CPU-Architecture](../Fundamental-Of-Operating-System/CPU-Architecture.md) — server CPU architecture and internals
- [Laptop-or-PC-Specifications](../Fundamental-Of-Operating-System/Laptop-or-PC-Specifications.md) — contrast with consumer hardware
