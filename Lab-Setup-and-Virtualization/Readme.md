# Lab Setup and Virtualization

Building the disposable, isolated environment where every technique in this course is practiced safely.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

You cannot learn infrastructure attack and defense on production. This module stands up a home/lab virtualization stack — VirtualBox, KVM/QEMU, or Proxmox — configures the networking modes that keep lab traffic contained, and sources legally obtained Windows media (evaluation ISOs, custom builds, activation) plus deliberately vulnerable machines to practice against. The result is a repeatable, snapshot-driven lab you can break and rebuild at will.

## Learning Objectives

By the end of this module you will be able to:

- Choose and set up a hypervisor (VirtualBox, KVM/QEMU, or Proxmox) for a Windows/AD lab
- Configure isolated virtual networks (NAT, host-only, internal) so lab traffic never touches production
- Obtain and prepare Windows evaluation media and vulnerable target machines for practice

## Topics Covered

This module contains **14 notes**.

| Note | Topic |
| --- | --- |
| [Virtualization](Virtualization.md) | Virtualization concepts and hypervisor types |
| [VirtualBox-Network-Modes](VirtualBox-Network-Modes.md) | NAT, host-only, internal, bridged networking |
| [KVM(Kernel-based-Virtual-Machine)](KVM(Kernel-based-Virtual-Machine).md) | KVM overview |
| [KVM-and-QEMU-Setup-on-Kali-Linux](KVM-and-QEMU-Setup-on-Kali-Linux.md) | Installing KVM/QEMU on Kali |
| [Proxmox-Setup](Proxmox-Setup.md) | Proxmox VE hypervisor setup |
| [Windows-Evaluation-Center](Windows-Evaluation-Center.md) | Legally sourcing Windows eval ISOs |
| [Custom-build-Windows-11-ISO](Custom-build-Windows-11-ISO.md) | Building a customized Windows 11 image |
| [Microsoft-Windows-Activation](Microsoft-Windows-Activation.md) | Windows activation in the lab |
| [Vulnerable-Machines](Vulnerable-Machines.md) | Deliberately vulnerable practice targets |
| [Hyper-V](Hyper-V.md) | Hyper-V hypervisor |
| [VMware-Workstation](VMware-Workstation.md) | VMware Workstation |
| [Virtual-Networking](Virtual-Networking.md) | Virtual networking modes |
| [Snapshots-and-Templates](Snapshots-and-Templates.md) | Snapshots and templates |
| [Lab-Design](Lab-Design.md) | Lab design and topology |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, build the baseline lab: a hypervisor host, one Windows Server (future DC), one Windows client, and one Kali attacker on an **isolated** internal network — then snapshot each VM clean before touching anything.

## Best Practices

- Snapshot every VM in a known-good state before each lab and roll back afterward
- Keep the lab on an isolated/host-only network with no route to production or the internet unless a lab requires it
- Use evaluation/eval-center media and rearm rather than pirated keys; document VM specs for reproducibility

## Security Considerations

> [!WARNING]
> **Contain the blast radius**
> Lab machines are intentionally weak and often run malware or offensive tooling — treat the whole environment as hostile to everything outside it.

- Never bridge a vulnerable target directly onto your home/office LAN
- Disable clipboard/shared-folder integration when detonating untrusted samples
- Rebuild from snapshot rather than "cleaning" a compromised lab host

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| VMs can't see each other but should | Wrong network mode — put lab VMs on the same host-only/internal network, not separate NATs |
| Nested VMs / KVM won't start | Hardware virtualization (VT-x/AMD-V) disabled in firmware, or nested virt off on the host |

## References

- [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/)
- [Proxmox VE documentation](https://pve.proxmox.com/pve-docs/)
- [VirtualBox networking modes](https://www.virtualbox.org/manual/ch06.html)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Fundamental of Operating System](../Fundamental-Of-Operating-System/Readme.md) — related module
- [Practical Labs](../Practical-Labs/Readme.md) — course-wide hands-on labs
