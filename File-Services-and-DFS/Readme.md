# File Services and DFS

Windows file-services tradecraft: physical and logical storage foundations, the NTFS permission model and its command-line tooling, and the file-server roles (DFS Namespaces, FSRM) that manage shared storage at enterprise scale.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

**File services** cover how Windows stores, protects, and shares data on disk. This module works bottom-up: from the physical media (internal disks, RAID arrays, tape) and file systems that organize them, through the NTFS discretionary access-control model and the `icacls`/`cacls`/`takeown` utilities that inspect and modify it, up to the server roles — DFS Namespaces and File Server Resource Manager — that present and govern shared storage across many servers. It also covers Alternate Data Streams, an NTFS feature with real offensive and defensive relevance.

## Learning Objectives

- Distinguish the common internal disk types, RAID levels, and tape media, and match them to durability and performance requirements.
- Explain the Windows file systems (FAT, NTFS, ReFS) and the on-disk structures that back NTFS.
- Read and modify NTFS permissions and ownership using the GUI, `icacls`, `cacls`, `takeown`, and PowerShell.
- Deploy and manage DFS Namespaces to abstract shared folders behind a single namespace.
- Enforce storage governance (quotas, file screening, classification, reporting) with FSRM.
- Detect and investigate Alternate Data Streams as a data-hiding and persistence technique.

## Topics Covered

This module contains **17 notes**.

| Note | Topic |
| --- | --- |
| [Types-of-Internal-Disks](Types-of-Internal-Disks.md) | Internal disk types — HDD, SSD, NVMe |
| [RAID-(Redundant-Array-of-Independent-Disks)](RAID-(Redundant-Array-of-Independent-Disks).md) | RAID levels and disk redundancy |
| [Tape-Storage](Tape-Storage.md) | Magnetic tape backup and archival storage |
| [File-System](File-System.md) | File-system fundamentals (FAT, NTFS, ReFS) |
| [NTFS-(New-Technology-File-System)-Permissions](NTFS-(New-Technology-File-System)-Permissions.md) | The NTFS permissions model |
| [NTFS-Default-Permissions](NTFS-Default-Permissions.md) | Default NTFS permission sets |
| [NTFS-Permissions-Setup-with-PowerShell](NTFS-Permissions-Setup-with-PowerShell.md) | Configuring NTFS permissions with PowerShell |
| [ICACLS-Command](ICACLS-Command.md) | Modern ACL management command |
| [CACLS-Command](CACLS-Command.md) | Legacy ACL command (predecessor to icacls) |
| [TAKEOWN-Command](TAKEOWN-Command.md) | Taking ownership of files and directories |
| [Alternate-Data-Streams(ADS)](Alternate-Data-Streams(ADS).md) | Hidden named data streams on NTFS |
| [DFS-Namespaces-(Distributed-File-System-Namespaces)](DFS-Namespaces-(Distributed-File-System-Namespaces).md) | Unified logical view of shared folders |
| [File-Server-Resource-Manager(FSRM)](File-Server-Resource-Manager(FSRM).md) | Storage quotas, file screening, and classification |
| [Share-Permissions](Share-Permissions.md) | SMB share permissions |
| [Storage-Quotas](Storage-Quotas.md) | Storage quotas with FSRM |
| [File-Screening](File-Screening.md) | File screening with FSRM |
| [DFS-Replication](DFS-Replication.md) | DFS Replication (DFSR) |

## Practical Labs

- Build a software RAID volume in Disk Management / Storage Spaces and simulate a disk failure.
- Set NTFS permissions on a share three ways — GUI, `icacls`, and PowerShell — and compare the resulting ACLs.
- Take ownership of a locked folder with `takeown` and re-grant access with `icacls`.
- Stand up a domain-based DFS Namespace with two folder targets and verify referral failover.
- Configure an FSRM quota and file screen, then confirm the block and notification fire.
- Hide an executable in an Alternate Data Stream, then enumerate it with `dir /r` and `Get-Item -Stream *`.

## Best Practices

- Assign NTFS permissions to groups, not individual users, and prefer least privilege.
- Let permissions inherit from a well-designed folder root; break inheritance only when necessary.
- Use RAID for availability, never as a substitute for backups.
- Front shared folders with a DFS Namespace so paths survive server migrations.
- Enforce quotas and file screens with FSRM to prevent uncontrolled storage growth.

## Security Considerations

- Overly broad ACLs (for example `Everyone: Full Control`) are a primary lateral-movement and data-exposure risk.
- Alternate Data Streams can conceal malware and exfiltrated data; include them in file-integrity monitoring.
- Taking ownership can silently override deny ACEs — audit `takeown`/ownership changes.
- DFS referrals and share permissions layer on top of NTFS; the most restrictive of the two wins.

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| "Access denied" despite a share grant | Restrictive NTFS ACL under a permissive share | Reconcile NTFS and share permissions; effective access is the most restrictive |
| Cannot modify a file's ACL | Current user is not the owner | Use `takeown` to seize ownership, then re-grant with `icacls` |
| DFS path unreachable | Broken folder target or referral | Check target health and enabled referrals in DFS Management |
| FSRM quota not enforced | Quota applied to the wrong path or soft quota | Verify the quota path and switch to a hard quota |
| Hidden data suspected on NTFS | Alternate Data Stream | Enumerate with `dir /r` or `Get-Item -Stream *` |

## References

- <https://learn.microsoft.com/en-us/windows-server/storage/dfs-namespaces/dfs-overview>
- <https://learn.microsoft.com/en-us/windows-server/storage/fsrm/fsrm-overview>
- <https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview>

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows Server Management](../Windows-Server-Management/Readme.md) — related module
- [Windows Operating System Administration](../Windows-Operating-System-Administration/Readme.md) — related module
