# File Server Resource Manager (FSRM)

File Server Resource Manager (FSRM) is a role service in Windows Server used to manage and classify data stored on file servers. It provides administrators with tools for storage quota management, file screening, file classification, automated file-management tasks, and reporting — helping organizations monitor disk usage, prevent unauthorized file storage, and automate file-management policies.

## Overview

FSRM sits on top of NTFS volumes and gives administrators central control over *what* users may store, *how much* they may store, and *what happens* to files over their lifecycle. Its five capability areas are quota management, file screening, file classification (FCI), file-management tasks, and storage reports.

## Concepts

### Quota Management

Quota management controls how much disk space users or folders can consume.

- Set storage limits on folders or volumes.
- Create **soft** (monitor/warn only) or **hard** (enforce) quotas.
- Generate warnings when limits are reached.
- Use quota **templates** for consistent, reusable deployment.

Example use cases:

- Limit department shared folders to 100 GB.
- Prevent users from consuming excessive storage.
- Send email alerts when usage exceeds thresholds.

> [!TIP]
> **Deep dive**
> Quotas are covered in detail in [Storage-Quotas](Storage-Quotas.md).

### File Screening Management

File screening restricts users from storing specific file types on the server. Administrators block files based on extensions such as `.mp3`, `.mp4`, `.exe`, and `.iso`.

```text
Block users from storing MP3 and video files in shared company folders.
```

Types of file screening:

| Type | Behavior |
| --- | --- |
| Active screening | **Blocks** restricted files from being saved |
| Passive screening | **Allows** files but logs the activity for auditing |

> [!TIP]
> **Deep dive**
> File screening is covered in detail in [File-Screening](File-Screening.md).

### File Classification Infrastructure (FCI)

FCI automatically classifies files according to rules and properties. Classification can be based on file content, location, type, metadata, or keywords.

Example classifications: **Confidential**, **Financial Data**, **HR Documents**, **Expired Files**.

Policies can then be applied automatically, such as encryption, retention policies, expiration rules, and access restrictions.

### File Management Tasks

FSRM can automate actions on files based on conditions or classifications:

- Delete expired files.
- Move files to archive folders.
- Encrypt sensitive files.
- Execute custom scripts.
- Generate notifications.

```text
Automatically delete temporary files older than 30 days.
```

### Storage Reports

FSRM provides detailed storage and usage reports:

- Disk usage reports
- Large files reports
- Duplicate files reports
- File screening audit reports
- Quota usage reports

These reports help administrators monitor storage growth, plan future capacity, detect unauthorized files, and improve storage efficiency.

## Installation

Install FSRM with PowerShell:

```powershell
Install-WindowsFeature -Name FS-Resource-Manager -IncludeManagementTools
```

### Installing FSRM from Server Manager

1. Open **Server Manager**.
2. Select **Add Roles and Features**.
3. Under **File and Storage Services → File and iSCSI Services**, choose **File Server Resource Manager**.
4. Complete the installation wizard, including the management tools when prompted.

> [!NOTE]
> **Screenshot**
> ![Add Roles and Features wizard with File Server Resource Manager checked under File and Storage Services](placeholder-screenshot)

## Administration

FSRM can be managed using:

- **FSRM MMC snap-in** (`fsrm.msc`) — the primary GUI console.
- **Windows PowerShell** — the `FileServerResourceManager` module.
- **Server Manager** — role installation and basic management.

> [!NOTE]
> **Screenshot**
> ![File Server Resource Manager console showing Quota Management, File Screening Management, Storage Reports Management, and Classification Management nodes](placeholder-screenshot)

## Enterprise Deployment

### File system support

FSRM supports **NTFS** volumes. FSRM does **not** support **ReFS** (Resilient File System).

### Supported Windows Server versions

FSRM is included in modern Windows Server editions: Windows Server 2008, 2012, 2016, 2019, and 2022.

### Common enterprise use cases

- Storage quota enforcement.
- Preventing media (and other unwanted) file storage.
- Data classification and compliance.
- Monitoring storage growth.
- Securing confidential files.
- Automating file lifecycle management.

## Security Considerations

- Use **active** file screening plus **passive** logging to both block and audit prohibited content.
- Combine FCI classification with automated encryption/retention to enforce data-handling policy.
- Quotas and screens apply to a path, not a user identity — verify they are set on the correct folder and are **hard** quotas where enforcement matters.
- FSRM does not protect ReFS volumes; keep governed shares on NTFS.

## Best Practices

- Build **quota** and **file-screen templates** once and apply them across shares for consistency.
- Prefer **auto-apply quotas** on parent folders so new subfolders inherit limits.
- Schedule **storage reports** to catch growth and unauthorized files early.
- Test screens/quotas in **passive/soft** mode before switching to active/hard enforcement.

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| Quota not enforced | Soft quota, or applied to the wrong path | Verify the quota path and switch to a hard quota |
| Users still saving blocked files | Passive screen, or file group missing the extension | Switch to active screening and confirm the blocked file group |
| FSRM unavailable on a volume | Volume is ReFS | Move governed data to an NTFS volume |
| Email notifications not sent | SMTP not configured in FSRM options | Set the SMTP server under FSRM → Configure Options |

## References

- Microsoft Learn — File Server Resource Manager overview: <https://learn.microsoft.com/en-us/windows-server/storage/fsrm/fsrm-overview>
- Microsoft Learn — FileServerResourceManager PowerShell module: <https://learn.microsoft.com/en-us/powershell/module/fileserverresourcemanager/>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Storage-Quotas](Storage-Quotas.md) — FSRM disk-space quota enforcement — related note
- [File-Screening](File-Screening.md) — blocking prohibited file types with FSRM — related note
- [DFS-Namespaces-(Distributed-File-System-Namespaces)](DFS-Namespaces-(Distributed-File-System-Namespaces).md) — companion file-services role — related note
