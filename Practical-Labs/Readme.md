# Practical Labs

The course-wide, hands-on lab collection — reproducible walkthroughs that turn each module's theory into muscle memory.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Reading about infrastructure is not the same as building and breaking it. This collection gathers guided, reproducible labs — each with a target, setup steps, an attack or configuration walkthrough, and the expected result — so every module graduates from personal study to something you can actually do on a disposable VM. Labs assume the baseline environment from the Lab Setup module and are grouped to follow the course arc from fundamentals through Active Directory to hardening.

## Learning Objectives

By the end of this collection you will be able to:

- Reproduce each module's core technique on an isolated lab from a documented walkthrough
- Verify success against a stated expected result rather than guessing
- Chain individual labs into the end-to-end scenarios in the Enterprise Projects module

## Topics Covered

This module contains **7 labs**.

| Lab | Focus |
| --- | --- |
| [Lab-01-Lab-Foundations](Lab-01-Lab-Foundations.md) | Hypervisor, isolated network, Windows/Kali VMs, snapshots |
| [Lab-02-Core-Services](Lab-02-Core-Services.md) | DNS, DHCP, IIS, FTP, File Services build-and-verify |
| [Lab-03-Active-Directory](Lab-03-Active-Directory.md) | DC promotion, OU/GPO, users/groups, trusts |
| [Lab-04-Remote-Access](Lab-04-Remote-Access.md) | RRAS VPN + NPS, RDP hardening |
| [Lab-05-Attack-and-Defense](Lab-05-Attack-and-Defense.md) | Kerberoasting, LLMNR poisoning, and the controls that stop them |
| [Lab-06-Backup-and-Recovery](Lab-06-Backup-and-Recovery.md) | System State / BMR / authoritative AD restore drills |
| [Lab-07-Monitoring](Lab-07-Monitoring.md) | Audit policy, Sysmon, event forwarding |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> This **is** the labs collection. Start from the baseline environment in [Lab Setup and Virtualization](../Lab-Setup-and-Virtualization/Readme.md), snapshot every VM clean, and work the tracks above in course order.

## Best Practices

- Snapshot before every lab and roll back after — labs are meant to be destructive and repeatable
- Follow the walkthrough once, then repeat it from memory to confirm the technique actually stuck
- Keep your own notes/screenshots of each run so results are reproducible and reportable

## Security Considerations

> [!WARNING]
> **Keep labs isolated**
> Labs run intentionally weak configurations and offensive tooling — never bridge them to a production or home network.

- Use host-only/internal networks; no route to the internet or corporate LAN unless a lab requires it
- Rebuild from snapshot rather than "cleaning" a machine you have attacked
- Never reuse lab credentials, keys, or certificates anywhere real

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Lab steps fail because a prior service is missing | Labs assume the module's build steps ran — complete the prerequisite module lab first |
| Attacker VM can't reach the target | Wrong virtual network mode — put both VMs on the same host-only/internal network |

## References

- [Lab Setup and Virtualization](../Lab-Setup-and-Virtualization/Readme.md) — building the baseline lab environment
- [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/) — legally sourced Windows media
- [Detection Lab / AD lab-building references](https://github.com/clong/DetectionLab)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Lab Setup and Virtualization](../Lab-Setup-and-Virtualization/Readme.md) — prerequisite environment
- [Enterprise Projects](../Enterprise-Projects/Readme.md) — end-to-end capstone scenarios
