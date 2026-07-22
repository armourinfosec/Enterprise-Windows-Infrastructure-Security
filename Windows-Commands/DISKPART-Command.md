# DISKPART Command

`diskpart` is the built-in Windows command-line disk-partitioning utility. It manages disks, partitions, and volumes — creating, deleting, formatting, extending, and assigning them — and is the scriptable successor to the older `fdisk`. It runs in its own interactive shell and operates on the object (disk, partition, or volume) currently "in focus".

## Overview

`diskpart` works on a focus model: you first **select** a disk, partition, or volume, and subsequent commands act on that selected object. It is used during Windows setup (partitioning before install), disk provisioning, USB/boot-media preparation, VHD attach/detach, and recovery. It must be run from an elevated command prompt.

> [!WARNING]
> `diskpart` operates directly on physical media. Commands such as `clean`, `delete`, and `format` are **irreversible and destroy data**. Always confirm the selected disk with `list disk` and `detail disk` before any write operation — selecting the wrong disk will wipe the wrong drive.

## Syntax

Enter the interactive shell, then issue commands one per line:

```cmd
diskpart
```

```text
DISKPART> list disk
DISKPART> select disk 1
DISKPART> clean
DISKPART> create partition primary
DISKPART> format fs=ntfs quick
DISKPART> assign letter=E
DISKPART> exit
```

Run a saved script non-interactively:

```cmd
diskpart /s C:\scripts\provision-disk.txt
```

## Parameters

| Command | Description |
| --- | --- |
| `list disk \| partition \| volume` | Enumerate disks, partitions (of selected disk), or volumes |
| `select disk N` | Put disk N in focus |
| `select partition N` | Put partition N (on the selected disk) in focus |
| `select volume N` | Put volume N in focus |
| `detail disk \| partition \| volume` | Show detailed properties of the focused object |
| `clean` | Remove all partitioning/formatting from the selected disk |
| `clean all` | Zero every sector on the disk (slow, secure wipe) |
| `create partition primary \| extended \| logical` | Create a partition on the selected disk |
| `format fs=ntfs \| fat32 \| exfat [quick] [label=...]` | Format the selected volume/partition |
| `assign letter=X` | Assign a drive letter to the focused volume |
| `remove letter=X` | Remove a drive letter |
| `active` | Mark the selected (MBR) partition active/bootable |
| `extend [size=N]` | Extend the selected volume into adjacent free space |
| `shrink desired=N` | Shrink the selected volume by N MB |
| `convert gpt \| mbr \| dynamic \| basic` | Convert the selected disk's style |
| `attributes disk clear readonly` | Clear the read-only attribute on the disk |
| `online disk` / `offline disk` | Bring the selected disk online/offline |
| `exit` | Leave the DISKPART shell |

## Examples

Inspect what is present before touching anything:

```text
DISKPART> list disk
DISKPART> select disk 1
DISKPART> detail disk
```

Wipe a disk and lay down a single NTFS partition with a drive letter:

```text
DISKPART> select disk 1
DISKPART> clean
DISKPART> create partition primary
DISKPART> format fs=ntfs quick label=DATA
DISKPART> assign letter=E
```

Prepare a bootable USB stick (MBR/FAT32, active):

```text
DISKPART> select disk 2
DISKPART> clean
DISKPART> create partition primary
DISKPART> select partition 1
DISKPART> active
DISKPART> format fs=fat32 quick
DISKPART> assign
```

Extend a volume into newly available space:

```text
DISKPART> select volume 3
DISKPART> extend
```

Clear a read-only flag that blocks writes:

```text
DISKPART> select disk 1
DISKPART> attributes disk clear readonly
```

Convert a freshly cleaned disk to GPT (required for UEFI system disks):

```text
DISKPART> select disk 1
DISKPART> clean
DISKPART> convert gpt
```

Attach and detach a virtual disk (VHD/VHDX):

```text
DISKPART> select vdisk file="C:\vm\data.vhdx"
DISKPART> attach vdisk
DISKPART> detach vdisk
```

Secure-wipe every sector (slow — hours on large HDDs):

```text
DISKPART> select disk 1
DISKPART> clean all
```

## Enterprise Usage

- **Automated provisioning**: `diskpart /s script.txt` partitions and formats disks unattended during OS deployment (MDT, WDS, SCCM task sequences).
- **Boot-media creation**: prepare USB installation drives with a repeatable clean/create/active/format sequence.
- **Storage servicing**: extend/shrink volumes, bring SAN/iSCSI LUNs online, and clear read-only attributes on newly attached disks.
- **VHD lifecycle**: attach/detach VHDX files for offline servicing of virtual machine disks.
- **Decommissioning**: `clean all` sanitizes drives before repurpose or disposal (note: for SSDs, manufacturer secure-erase is preferred).

## Security Considerations

> [!IMPORTANT]
> `diskpart` requires Administrator rights and is a destructive-capability tool. Restrict who can run it and log its use — an attacker with disk-level access can wipe volumes (`clean`), destroy evidence, or attach a rogue VHD.

- `clean` / `clean all` are anti-forensic wiping primitives; monitor for `diskpart` execution on servers and endpoints.
- Marking a partition `active` or converting boot disks can render a system unbootable — a denial-of-service vector.
- Bringing a disk `online` and clearing `readonly` can expose data on disks that policy intended to keep offline/read-only.
- Because it bypasses the recycle bin and file-level ACLs entirely, always double-check `list disk` output against `detail disk` (serial/size) before writing.

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| `Access is denied.` | Not running elevated | Reopen the command prompt as Administrator |
| `The disk is write-protected.` | Read-only attribute or physical write-lock | `attributes disk clear readonly`; check any hardware write-protect switch |
| `Virtual Disk Service error: The volume cannot be extended...` | No contiguous free space after the volume | Ensure unallocated space is immediately adjacent, or use dynamic disk |
| `Extend` fails on the system volume | Recovery partition sits between the volume and free space | Remove/relocate the intervening partition or use third-party tooling |
| Selected disk shows "no fixed disks" | Disk offline or not initialized | `online disk`, then `convert gpt`/`mbr` to initialize |
| Wrong drive letter conflict | Letter already in use | `remove letter=X` on the conflicting volume, then reassign |

## References

- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/diskpart>
- <https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/diskpart-command-line-options>
- <https://learn.microsoft.com/en-us/windows-server/storage/disk-management/overview-of-disk-management>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [BCDEDIT-Command](BCDEDIT-Command.md) — boot configuration data editing after partitioning — related note
- [Windows-Advanced-Boot-Options](Windows-Advanced-Boot-Options.md) — recovery and boot options that depend on the active/boot partition — related note
