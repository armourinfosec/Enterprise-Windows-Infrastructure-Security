# File System

## Overview

A **file system** is the method and data structure an operating system uses to organize, store, retrieve, and secure files on a storage device. It manages metadata (timestamps, size, permissions), allocates disk space, and enforces data integrity and access control — effectively the indexing system for everything stored on a hard drive, SSD, or USB device.

## Concepts

### Key functions of a file system

| Function | Description |
|---|---|
| File organization | Organizes data into folders and subfolders |
| Metadata handling | Stores information like file size, type, created/modified times |
| Access control | Manages user permissions (read/write/execute) |
| Space management | Allocates space on the disk, handles fragmentation |
| Fault tolerance | Prevents or recovers from file corruption (e.g., journaling in NTFS/ReFS) |

### Storage media that use file systems

- **Internal drives** (HDDs, SSDs)
- **External storage** (USB drives, memory cards)
- **Network-attached storage** (NAS)
- **Virtual disks** (VHD, VHDX)
- **Cloud-mounted storage** (via services like OneDrive)

### Example: file path in Windows

```text
C:\Users\Rahul\Documents\report.docx
```

- `C:` = Drive
- `\Users\Rahul\Documents\` = Folder path
- `report.docx` = File name

All of this structure is managed by the file system (usually **NTFS** in Windows).

## Types of File Systems in Windows

### 1. FAT32 (File Allocation Table 32)

- **Highly compatible** with Windows, Linux, macOS, and devices like TVs and cameras.
- **File size limit**: 4 GB max per file.
- **Partition size limit**: 2 TB.
- No support for permissions or encryption.
- Commonly used in **USB flash drives**, **memory cards**, and older systems.

### 2. NTFS (New Technology File System)

- **Default file system** for modern Windows (from Windows XP onward).
- Supports **large files and partitions**.
- Supports:
  - File and folder **permissions (ACLs)**
  - **Compression**
  - **Encryption (EFS)**
  - **Journaling** for crash recovery
- Used on **internal drives**, system partitions, and business environments.

### 3. exFAT (Extended File Allocation Table)

- Designed for **external storage**, especially large-capacity USB drives and SDXC cards.
- Supports **files larger than 4 GB**.
- Compatible with **Windows, macOS**, and **Linux (with drivers)**.
- No native journaling or permission system.
- Ideal for cross-platform file transfers.

### 4. ReFS (Resilient File System)

- Introduced in **Windows Server 2012**.
- Built for **fault tolerance**, **auto-repair**, and **high resilience**.
- Excellent for large volumes and virtualized workloads (e.g., Hyper-V).
- Not widely used in consumer systems.
- Best suited for **enterprise and server** environments.

### Feature comparison

| Feature | FAT32 | NTFS | exFAT | ReFS |
|---|---|---|---|---|
| Max file size | 4 GB | ~16 TB+ | 16 EB | 35 PB+ |
| Max volume size | 2 TB | 256 TB (Windows) | 128 PB | 35 PB+ |
| Journaling | No | Yes | No | Yes |
| Permissions (ACLs) | No | Yes | No | Yes |
| Encryption support | No | Yes (EFS) | No | BitLocker only |
| OS compatibility | All OSes | Windows only | Win/macOS/Linux | Windows Server only |
| Usage | USB, legacy | Internal disks | External drives | Server storage |

## GUI Steps

To check a volume's file system in **File Explorer**:

1. Open **This PC / My Computer**.
2. Right-click a drive → **Properties**.
3. Check the **File system** field.

> [!NOTE]
> **Screenshot**
> ![Drive Properties dialog with the General tab showing the File system field reading NTFS](placeholder-screenshot)

## Examples

### Checking a volume's file system from Command Prompt

```cmd
fsutil fsinfo volumeinfo C:
```

Output example:

```text
File System Name : NTFS
Serial Number : 0x12345678
Supports Case-sensitive : FALSE
Preserves Case of File Names : TRUE
Supports Unicode in File Names : TRUE
```

## Best Practices

| Scenario | Recommended file system |
|---|---|
| USB drive under 4 GB | FAT32 |
| USB with files larger than 4 GB | exFAT |
| Internal Windows OS drive | NTFS |
| Server backup or large storage | ReFS |
| macOS + Windows compatibility | exFAT |

- Use **NTFS** for any volume that must enforce access control, as FAT32 and exFAT store no permissions.
- Prefer **ReFS** for large virtualization and backup repositories where auto-repair matters.

## Security Considerations

- Only **NTFS** and **ReFS** enforce ACLs; data on FAT32/exFAT volumes is readable by anyone with physical access.
- Combine NTFS with **EFS** or **BitLocker** to protect data at rest.
- NTFS journaling aids recovery but also leaves forensic artifacts (e.g., the USN journal, `$MFT`) that record file activity.

## Troubleshooting

| Symptom | Likely cause | Resolution |
|---|---|---|
| "File is too large for the destination file system" | Copying a file over 4 GB to a FAT32 volume | Reformat the target as exFAT or NTFS |
| Permissions tab missing in drive properties | Volume is FAT32/exFAT, which lack ACLs | Convert or reformat to NTFS |
| Drive shows as RAW / unformatted | Corrupted or unrecognized file system | Run `chkdsk`, or recover data before reformatting |

## References

- <https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview>
- <https://learn.microsoft.com/en-us/windows-server/storage/refs/refs-overview>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/fsutil>

## Related

- [NTFS-(New-Technology-File-System)-Permissions](NTFS-(New-Technology-File-System)-Permissions.md) — NTFS is the default Windows file system
- [Types-of-Internal-Disks](Types-of-Internal-Disks.md) — file systems live on these physical disks
- [NTFS-Default-Permissions](NTFS-Default-Permissions.md) — default ACLs that ship with NTFS volumes
- [RAID-(Redundant-Array-of-Independent-Disks)](RAID-(Redundant-Array-of-Independent-Disks).md) — file systems run on top of RAID volumes
- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
