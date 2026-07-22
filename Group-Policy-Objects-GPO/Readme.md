# Group Policy Objects (GPO)

The Active Directory feature that centrally enforces OS, security, and user-restriction settings across every domain-joined computer and user.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Group Policy lets an administrator define settings once and push them to thousands of machines — password policy, software restriction, drive mappings, firewall rules, and much more. This module covers what a GPO is and how it is processed, the built-in Default Domain Policy, how to link and scope policy in a domain, and a concrete hardening example: blocking PowerShell for standard users. It also frames the defender's angle — common ways users leak data and how policy reduces that surface.

## Learning Objectives

By the end of this module you will be able to:

- Explain what a GPO is and how policy is inherited and applied (site → domain → OU)
- Create, link, and scope domain-based Group Policy
- Apply a security-hardening GPO such as restricting PowerShell for standard users

## Topics Covered

This module contains **11 notes**.

| Note | Topic |
| --- | --- |
| [Group-Policy(GPO)](Group-Policy(GPO).md) | Group Policy overview |
| [Default-Domain-Policy](Default-Domain-Policy.md) | The built-in Default Domain Policy GPO |
| [Domain-Based-Group-Policy-Configuration](Domain-Based-Group-Policy-Configuration.md) | Linking and scoping policy in a domain |
| [PowerShell-Blocking-Using-Group-Policy](PowerShell-Blocking-Using-Group-Policy.md) | Restricting PowerShell via Group Policy |
| [10-Common-Ways-Users-Leak-Data](10-Common-Ways-Users-Leak-Data.md) | Data-leakage vectors policy can reduce |
| [Administrative-Templates](Administrative-Templates.md) | Administrative Templates (ADMX) |
| [GPO-Processing-Order](GPO-Processing-Order.md) | GPO processing order (LSDOU) |
| [Loopback-Processing](Loopback-Processing.md) | Loopback processing mode |
| [WMI-Filters](WMI-Filters.md) | WMI filters for GPO targeting |
| [Software-Deployment-via-GPO](Software-Deployment-via-GPO.md) | Software deployment via GPO |
| [GPO-Troubleshooting](GPO-Troubleshooting.md) | Troubleshooting Group Policy |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, create a test OU, link a GPO that enforces a password policy or blocks the PowerShell executable, move a client into the OU, and confirm application with `gpresult /r` and `gpupdate /force`.

## Best Practices

- Never edit the Default Domain Policy for anything but account/password policy — create purpose-built GPOs instead
- Scope tightly with OUs, security-group filtering, and WMI filters rather than broad domain links
- Name GPOs descriptively and document their intent; use `gpresult`/RSoP to verify effective settings

## Security Considerations

> [!WARNING]
> **GPO is a two-way street**
> The same mechanism that hardens endpoints is a prime lateral-movement and persistence vector — an attacker with GPO edit rights can push code domain-wide.

- Restrict who can create and link GPOs; audit changes to policy objects
- Use GPO to disable legacy protocols, enforce LAPS, and constrain scripting hosts
- Remember GPO restrictions on interpreters (PowerShell) are a speed bump, not a boundary — pair with AppLocker/WDAC

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| A GPO isn't applying to a machine | Scoping/inheritance issue — run `gpresult /h report.html` to see which GPOs win and why; check security filtering and Block Inheritance |
| Settings apply late or inconsistently | Replication or slow-link detection — force with `gpupdate /force` and verify DC replication |

## References

- [Group Policy overview (Microsoft Learn)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831791(v=ws.11))
- [Security baselines / Microsoft Security Compliance Toolkit](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10)
- [MITRE ATT&CK — Domain Policy Modification (T1484)](https://attack.mitre.org/techniques/T1484/)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Active Directory Domain Services (AD DS)](../Active-Directory-Domain-Services-AD-DS/Readme.md) — related module
- [Windows Operating System Administration](../Windows-Operating-System-Administration/Readme.md) — related module
