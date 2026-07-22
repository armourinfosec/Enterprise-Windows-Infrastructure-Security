# Windows Operating System Administration

Day-to-day administration of a Windows host — local users and groups, profiles, auditing, and the event logs that record it all.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

This module covers the local administration of a Windows machine: managing users and groups (GUI, command line, and PowerShell), the built-in Administrator account and SIDs, the Guest account, profile and application data locations (`AppData`/`ProgramData`), and the visibility side — audit policy and the Windows Event Logs that record authentication, privilege use, and object access. These are the primitives that both an administrator and an incident responder work with.

## Learning Objectives

By the end of this module you will be able to:

- Manage local users and groups via the GUI, command line, and PowerShell
- Explain the built-in Administrator/Guest accounts, SIDs, and profile/data locations
- Configure audit policy and read Windows Event Logs to reconstruct activity

## Topics Covered

This module contains **9 notes**.

| Note | Topic |
| --- | --- |
| [User-Management](User-Management.md) | Managing local users and groups (GUI) |
| [User-Management-Command](User-Management-Command.md) | User/group management from the command line |
| [PowerShell-User-Group-Management](PowerShell-User-Group-Management.md) | User/group management with PowerShell |
| [Add-User-to-Administrators](Add-User-to-Administrators.md) | Granting local admin rights |
| [Windows-Local-Administrator-Account-and-SID](Windows-Local-Administrator-Account-and-SID.md) | Built-in Administrator account and SIDs |
| [Enable-Guest-Login](Enable-Guest-Login.md) | The Guest account |
| [AppData-and-ProgramData](AppData-and-ProgramData.md) | Profile and application data locations |
| [Windows-Audit-Policy](Windows-Audit-Policy.md) | Configuring audit policy |
| [Windows-Event-Logs](Windows-Event-Logs.md) | Reading and interpreting event logs |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, create a local user three ways (GUI, `net user`, PowerShell), add it to Administrators, enable an audit subcategory, then trigger and locate the corresponding events in Event Viewer.

## Best Practices

- Use standard accounts for daily work; reserve local Administrator for break-glass and rename/disable where policy allows
- Turn on advanced audit policy for logon, privilege use, and object access — you cannot investigate what you never logged
- Forward event logs to a central collector so a wiped host doesn't erase the evidence

## Security Considerations

> [!WARNING]
> **Local admin is the first rung of the ladder**
> Local administrator on one machine is often the pivot to credential theft and domain compromise — guard membership tightly.

- The well-known Administrator SID (`...-500`) is a standing target; rename, disable, or protect it with LAPS
- Event logs are an attacker's cleanup target — monitor for `1102` (log cleared) and ship logs off-host
- `AppData`/`ProgramData` are common persistence and staging locations — know them for both hardening and hunting

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Expected security events aren't logged | Audit subcategory not enabled — set it via `auditpol /set` or Advanced Audit Policy GPO |
| Can't add a user to a group | Insufficient rights or the account is domain- vs local-scoped — run elevated and target the correct SAM/domain |

## References

- [Manage local users and groups (Microsoft Learn)](https://learn.microsoft.com/en-us/windows/client-management/client-tools/manage-local-users-groups)
- [Windows security auditing (Microsoft Learn)](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/security-auditing-overview)
- [Local Administrator Password Solution (LAPS)](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows Server Management](../Windows-Server-Management/Readme.md) — related module
- [Windows Monitoring and Logging](../Windows-Monitoring-and-Logging/Readme.md) — related module
- [Group Policy Objects (GPO)](../Group-Policy-Objects-GPO/Readme.md) — related module
