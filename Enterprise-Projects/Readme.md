# Enterprise Projects

Ten end-to-end capstone builds that assemble the whole course into a working, defended enterprise Windows environment.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Modules teach one service at a time; real infrastructure is those services working together. This deliverable is a set of ten scenario-driven projects that each combine several modules into a complete objective — stand up a domain, publish services, connect remote users, harden the estate, then attack and defend it. Every project states a goal, prerequisites, a build sequence, and a definition of done, so finishing them demonstrates end-to-end capability rather than isolated knowledge.

## Learning Objectives

By the end of this deliverable you will be able to:

- Combine multiple modules (AD, DNS/DHCP, IIS, remote access, backup) into a working environment
- Execute a project from a goal statement, prerequisites, and a build sequence to a verified end state
- Attack a self-built environment and then apply the hardening that closes the gaps

## Topics Covered

This module contains **10 projects**.

| # | Project | Combines |
| --- | --- | --- |
| 1 | [Project-01-Single-DC-Domain](Project-01-Single-DC-Domain.md) | AD DS, DNS, OU/GPO |
| 2 | [Project-02-Core-Network-Services](Project-02-Core-Network-Services.md) | DHCP, DNS, networking |
| 3 | [Project-03-Publish-Web-and-Database](Project-03-Publish-Web-and-Database.md) | IIS, PHP/MySQL, SDLC |
| 4 | [Project-04-File-Services-and-DFS](Project-04-File-Services-and-DFS.md) | File Services, permissions |
| 5 | [Project-05-Remote-Access-for-a-Branch](Project-05-Remote-Access-for-a-Branch.md) | RRAS VPN, NPS, RDP |
| 6 | [Project-06-Backup-and-Disaster-Recovery](Project-06-Backup-and-Disaster-Recovery.md) | Backup/BMR, AD restore |
| 7 | [Project-07-Monitoring-and-Detection-Pipeline](Project-07-Monitoring-and-Detection-Pipeline.md) | Audit policy, Sysmon, WEF |
| 8 | [Project-08-Harden-the-Enterprise](Project-08-Harden-the-Enterprise.md) | Baselines, LAPS, tiering |
| 9 | [Project-09-Attack-the-Lab](Project-09-Attack-the-Lab.md) | Kerberoasting, poisoning, lateral movement |
| 10 | [Project-10-Purple-Team-Capstone](Project-10-Purple-Team-Capstone.md) | Attack + detect + remediate end to end |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> These projects are the largest labs in the course. Build them on the baseline environment from [Lab Setup and Virtualization](../Lab-Setup-and-Virtualization/Readme.md), reusing the module-level exercises in [Practical Labs](../Practical-Labs/Readme.md) as building blocks.

## Best Practices

- Snapshot at each project milestone so you can branch or roll back without rebuilding from scratch
- Document each project as you go (topology, credentials, decisions) as if it were an engagement report
- Finish a project only when its stated "definition of done" verifies — don't declare success on a partial build

## Security Considerations

> [!WARNING]
> **Build it, then break it — in isolation**
> The later projects deliberately attack the environment you built; keep the whole thing on an isolated network and treat it as hostile to everything outside.

- Never expose a project environment (especially the attack phases) to a production or home LAN
- Rotate/retire all project credentials and certificates; never reuse them in real systems
- Rebuild from snapshot after the attack projects rather than trusting a "cleaned" host

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| A project step assumes a service that isn't there | Prerequisite project/module not completed — build the dependency first (each project lists prerequisites) |
| End-state verification fails | Work back through the build sequence checkpoints; confirm DNS/AD health with `dcdiag` / `repadmin` before higher layers |

## References

- [Practical Labs](../Practical-Labs/Readme.md) — the module-level exercises these projects assemble
- [Microsoft Learn — Windows Server](https://learn.microsoft.com/en-us/windows-server/)
- [MITRE ATT&CK — Enterprise matrix](https://attack.mitre.org/matrices/enterprise/) — framing for the attack/defense projects

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Practical Labs](../Practical-Labs/Readme.md) — course-wide hands-on labs
- [Lab Setup and Virtualization](../Lab-Setup-and-Virtualization/Readme.md) — prerequisite environment
- [Enterprise Security](../Enterprise-Security/Readme.md) — the hardening projects build on this module
