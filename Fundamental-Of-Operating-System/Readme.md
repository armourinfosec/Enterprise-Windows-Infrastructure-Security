# Fundamental of Operating System

The computer, firmware, boot, and OS basics that everything else in the course builds on.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Before administering — or attacking — Windows infrastructure, you need a solid mental model of what runs beneath it. This module builds that foundation from the hardware up: CPU architecture and PC specifications, the firmware and boot chain (BIOS/UEFI, Secure Boot, the Windows Boot Manager, and the full power-on-to-logon sequence), and the operating-system concepts, editions, and release history of the Windows family. It closes with the note-taking tooling you'll use to document lab work throughout the course.

## Learning Objectives

By the end of this module you will be able to:

- Describe core computer hardware and read CPU/RAM/storage specifications
- Trace the Windows boot chain from firmware (BIOS/UEFI, Secure Boot) through the Boot Manager to logon
- Distinguish Windows client vs server editions and place releases on the product timeline

## Topics Covered

This module contains **11 notes**.

| Note | Topic |
| --- | --- |
| [Fundamental-Of-Computers](Fundamental-Of-Computers.md) | What a computer is and how its parts fit together |
| [Laptop-or-PC-Specifications](Laptop-or-PC-Specifications.md) | Reading and comparing hardware specs |
| [CPU-Architecture](CPU-Architecture.md) | CPU internals, 32-bit vs 64-bit, execution |
| [Firmware](Firmware.md) | Firmware between hardware and OS |
| [BIOS-and-UEFI](BIOS-and-UEFI.md) | Legacy BIOS vs modern UEFI, Secure Boot |
| [Windows-Boot-Manager](Windows-Boot-Manager.md) | The bootloader handing off to the kernel |
| [Booting-Process](Booting-Process.md) | Power-on to logon boot sequence |
| [Operating-System](Operating-System.md) | Processes, memory, hardware abstraction |
| [Windows-Operating-System-Editions](Windows-Operating-System-Editions.md) | Client vs server editions |
| [Windows-Operating-Systems-Timeline](Windows-Operating-Systems-Timeline.md) | Windows release history |
| [Note-Taking-Tools](Note-Taking-Tools.md) | Documenting lab work and findings |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, inspect your own lab machine: read the UEFI/Secure Boot state (`msinfo32` → BIOS Mode), enumerate the boot entries with `bcdedit /enum`, and confirm the OS edition/build with `winver`.

## Best Practices

- Keep UEFI + Secure Boot enabled on lab and production machines unless a test explicitly requires disabling them
- Match the OS edition to the workload (Server for infrastructure roles, not a client SKU)
- Record host and VM specs in your notes so labs are reproducible

## Security Considerations

> [!WARNING]
> **The boot chain is a trust anchor**
> Everything above it inherits the firmware's trust — a compromised boot chain undermines every OS-level control.

- Secure Boot and firmware passwords defend against bootkits and unauthorized boot media
- Legacy BIOS/MBR lacks Secure Boot — prefer UEFI/GPT for new builds
- Treat firmware updates like any other patch: source them from the vendor and verify integrity

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Machine boots to firmware instead of Windows | Boot order or missing boot entry — check UEFI boot menu and rebuild with `bcdedit` / `bootrec /rebuildbcd` |
| OS won't install / "Secure Boot" errors | Firmware/media mismatch — align UEFI vs Legacy mode with GPT vs MBR media |

## References

- [Windows boot process (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-hardware/drivers/bringup/boot-and-uefi)
- [UEFI Secure Boot (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-secure-boot)
- [Windows release information](https://learn.microsoft.com/en-us/windows/release-health/release-information)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows Operating System Administration](../Windows-Operating-System-Administration/Readme.md) — related module
- [Lab Setup and Virtualization](../Lab-Setup-and-Virtualization/Readme.md) — related module
