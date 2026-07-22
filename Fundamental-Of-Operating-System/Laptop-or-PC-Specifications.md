# Laptop or PC Specifications

Recommended laptop / PC specifications for a hacking or cybersecurity workstation, plus a reference build. These specs prioritise virtualization headroom (multiple VMs), fast storage, and strong CPU/RAM for security tooling.

## Overview

A security workstation is judged less by raw gaming performance and more by how many lab machines it can run at once and how quickly it moves data. A pentest lab typically runs several [operating systems](Operating-System.md) side by side — an attacker VM (Kali/Parrot), one or more Windows targets, and often a Domain Controller — so **CPU cores**, **RAM**, and **SSD speed** matter far more than a top-tier GPU.

Reading a spec sheet builds on the hardware fundamentals in [Fundamental-Of-Computers](Fundamental-Of-Computers.md) and [CPU-Architecture](CPU-Architecture.md); the firmware and boot behaviour those specs enable is covered in [Firmware](Firmware.md), [BIOS-and-UEFI](BIOS-and-UEFI.md), and [Booting-Process](Booting-Process.md). This note focuses on choosing and comparing the components themselves.

> [!TIP]
> **What to prioritise**
> If the budget is fixed, spend it in this order for a security workstation: **RAM → fast NVMe SSD → CPU cores → GPU**. Running out of RAM or disk stalls a whole lab; a slower GPU only slows one specialised task (password cracking).

## Recommended Specifications

- **Processor (CPU)** — Intel Core i5 / i7 or AMD Ryzen 5 / 7 (or better). Needed for running multiple virtual machines (VMs) and heavy security tools efficiently. Prefer more physical cores and confirm hardware **virtualization support** (Intel VT-x / AMD-V) is present and enabled in firmware.
- **RAM** — minimum 16 GB, preferably 32 GB. Allows multitasking, running multiple VMs at once, and smooth operation of penetration-testing tools.
- **Storage** — at least 512 GB SSD, 1 TB preferred. SSDs (ideally NVMe) provide faster boot times and quick loading of VMs and software, plus room for dual-boot installs and evidence files.
- **GPU** — optional dedicated GPU (NVIDIA GTX / RTX). Useful for password cracking or AI/ML-based security analysis, but not mandatory for every hacking class.
- **Display** — Full HD (1920x1080) or higher. Screen real estate helps when juggling multiple terminals and monitoring tools.
- **Operating System** — Windows with a dual-boot or VM running Kali Linux or another security-focused Linux distro. Kali Linux and Parrot OS are standard for ethical-hacking labs.
- **Ports** — multiple USB 3.0/3.1 ports, HDMI, and Ethernet (RJ-45), important for networking, external drives, and hardware exploits.
- **Battery life** — 6–8+ hours for portability during classes or fieldwork.
- **Build quality** — durable chassis with a good cooling system to withstand long lab sessions and prevent thermal throttling under sustained load.

### Summary Table

| Component | Recommended Specification |
| :-- | :-- |
| CPU | Intel i5/i7/i9 (10th Gen or above) / Ryzen 7 or above |
| RAM | 16 GB minimum, 32 GB preferred |
| Storage | 512 GB SSD minimum, 1 TB SSD preferred |
| GPU | Dedicated GPU (NVIDIA GTX/RTX) optional |
| Display | Full HD (1920x1080) or higher |
| OS | Windows 10/11 + Kali Linux dual-boot or VM |
| Ports | USB 3.0/3.1, HDMI, Ethernet |
| Battery Life | 6–8 hours or more |
| Build | Durable with efficient cooling |

These specs ensure smooth virtualization, effective penetration testing, network analysis, and malware simulation as needed in hacking labs.

## Reading Your Own Specs

Before buying or benchmarking, confirm what a machine already has and whether it can host VMs.

On Windows, the built-in tools report CPU, RAM, and firmware mode:

```cmd
systeminfo
msinfo32
winver
```

The same information, plus virtualization state, from PowerShell:

```powershell
Get-ComputerInfo -Property CsName,CsProcessors,CsTotalPhysicalMemory,OsName,BiosFirmwareType
```

On a Linux workstation (for example, a Kali host):

```bash
lscpu            # CPU model, cores, virtualization (VT-x/AMD-V) flags
free -h          # total and available RAM
lsblk            # storage devices and partitions
```

Confirm the CPU exposes hardware virtualization (needed for fast VMs):

```bash
grep -E -c '(vmx|svm)' /proc/cpuinfo   # non-zero means VT-x/AMD-V present
```

> [!NOTE]
> **Virtualization must be enabled in firmware**
> A CPU can support VT-x / AMD-V while the feature is switched **off** in BIOS/UEFI. If VMs refuse to start or run only in slow emulation, enable virtualization in firmware settings. See [BIOS-and-UEFI](BIOS-and-UEFI.md).

## Comprehensive Specification of My Build

A reference high-end desktop build suited to heavy virtualization, password cracking, and general lab work.

| Component | Model/Specification | Key Features / Description |
| :-- | :-- | :-- |
| CPU | **AMD Ryzen 9 7950X3D 16-Core** | 16 cores, 32 threads, base clock 4.2 GHz, boost up to 5.7 GHz, 128 MB L3 cache, 120 W TDP, AM5 socket |
| Motherboard | **MSI MEG X670E ACE** | E-ATX, 22+2+1 power phases, DDR5 dual channel up to 6666+ MHz (OC), PCIe 5.0, 6x M.2, advanced cooling, 10G LAN, Wi-Fi 6E |
| RAM | **G.Skill Trident Z5 NEO RGB 64GB (2x32GB) DDR5-6000** | DDR5, 6000 MHz, CL30-36-36-96, AMD EXPO, RGB, dual channel |
| Storage | **Samsung 990 PRO SSD 2TB PCIe 4.0 M.2** | PCIe 4.0, M.2 2280, up to 7450 MB/s read, 6900 MB/s write, 600 TBW, 5-year warranty |
| CPU Cooler | **Corsair iCUE H150i ELITE LCD XT Liquid** | 360 mm radiator, LCD display, quiet & extreme cooling modes, iCUE software control, suitable for high-TDP CPUs |
| Power Supply | **Corsair HX1000i ATX 3.0 80 Plus Platinum** | Fully modular, 1000 W, PCIe 5.0 support, 92% platinum efficiency, FDB fan, zero-RPM mode, Japanese capacitors |
| Chassis | **Gamdias NESO P1 RB (Dual-Chamber Full Tower)** | E-ATX support, up to 426 mm GPU, up to 360/420 mm radiators, up to 8 expansion slots, multiple drive bays |
| Case Fans | **AEOLUS M2-1204R** | 120 mm, ARGB, trio-loop lighting, 600–1500 RPM, 56 CFM airflow, hydraulic bearing, remote & sync box |
| GPU | **NVIDIA GeForce RTX 4070 Founder's Edition** | Ada Lovelace, 12 GB GDDR6X, 220–235 W TDP, 7800+ CUDA cores, PCIe 4.0, ray tracing & DLSS 3.0 |
| UPS | **Microtek UPS Legend 1600** | Offline/line-interactive UPS, 1600 VA, surge protection (for high-wattage PC + peripherals) |

### Notes

- This configuration supports top-tier gaming, content creation, virtualization, and is suitable for overclocking and advanced cooling.
- All listed components are compatible for a modern, high-performance build with PCIe 5.0, DDR5, and future-proof power delivery.
- Additional accessories such as the ARGB case fans and the UPS provide aesthetics, cooling, and power safety.

## Security Considerations

Hardware choices have direct offensive and defensive consequences.

- **Password cracking (offensive)** — a strong CUDA/OpenCL GPU dramatically accelerates hashcat-style attacks; this is the one task where the GPU, not the CPU, is the bottleneck.
- **Isolated lab (defensive discipline)** — abundant RAM and cores let you detonate malware and run vulnerable targets inside VMs instead of on the host, containing the blast radius.
- **Engagement data at rest** — a pentest laptop routinely holds client scope documents, screenshots, and captured credentials.

> [!WARNING]
> **Protect data on the workstation itself**
> A pentester's laptop is a high-value target. Enable **full-disk encryption** (BitLocker on Windows, LUKS on Linux), keep firmware and Secure Boot enabled (see [BIOS-and-UEFI](BIOS-and-UEFI.md)), and never store client engagement data unencrypted or mixed with personal files. Loss or theft of an unencrypted machine can expose an entire client's attack surface.

## Best Practices

- Buy **more RAM than you think you need** — VMs consume it fastest; 32 GB is a practical floor for multi-VM AD labs.
- Prefer an **NVMe SSD** over SATA; VM disk I/O is often the real performance ceiling.
- Confirm and **enable virtualization** (VT-x / AMD-V) in firmware before building a lab.
- Keep the **host OS clean**: run tools and targets inside VMs, snapshot before risky work, and revert afterwards.
- Record host and VM specs in your notes (see [Note-Taking-Tools](Note-Taking-Tools.md)) so labs are reproducible.

## Troubleshooting

| Symptom | Likely cause & fix |
| :-- | :-- |
| VMs won't start or run extremely slowly | Virtualization disabled in firmware — enable VT-x / AMD-V in [BIOS-and-UEFI](BIOS-and-UEFI.md) settings |
| Host freezes when several VMs run | RAM exhaustion — reduce per-VM memory, run fewer VMs, or add RAM |
| Slow VM boot and file operations | Storage bottleneck — move VM disks to an NVMe SSD instead of a spinning disk |
| GPU not used for password cracking | Missing/incorrect vendor drivers (CUDA/OpenCL) — install the GPU vendor's compute drivers |

## References

- [Windows system information tools (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/systeminfo)
- [Hyper-V hardware requirements (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/system-requirements-for-hyper-v-on-windows)
- [Kali Linux system requirements (Kali Docs)](https://www.kali.org/docs/installation/hard-disk-install/)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Fundamental-Of-Computers](Fundamental-Of-Computers.md) — how the parts of a computer fit together
- [CPU-Architecture](CPU-Architecture.md) — CPU internals behind the spec sheet
- [Firmware](Firmware.md) — firmware layer between hardware and OS
- [BIOS-and-UEFI](BIOS-and-UEFI.md) — where virtualization and Secure Boot are configured
- [Operating-System](Operating-System.md) — the OS running on this hardware
- [Note-Taking-Tools](Note-Taking-Tools.md) — recording host and VM specs for reproducible labs
