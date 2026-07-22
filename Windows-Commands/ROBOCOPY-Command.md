# ROBOCOPY Command

`robocopy` (**Robust File Copy**) is a resilient, multithreaded command-line file-replication tool built into Windows. It is designed for reliable copying of large directory trees, resuming interrupted transfers, mirroring folders, and preserving NTFS attributes and permissions — capabilities that the older `copy` and `xcopy` commands lack.

## Overview

`robocopy` copies **directories**, not individual files by name in the classic sense — you specify a *source* folder, a *destination* folder, and an optional *file mask*. It retries on failure, can run multiple copy threads, preserves timestamps/attributes/ACLs, and returns a bitmap exit code describing what happened.

```cmd
robocopy <source> <destination> [file [file]...] [options]
```

> [!WARNING]
> The `/MIR` (mirror) option makes the destination an **exact** copy of the source — it **deletes** files and folders in the destination that no longer exist in the source. Point `/MIR` at the wrong destination and you can erase data. Test with `/L` (list only) first.

## Syntax

```cmd
robocopy /?
```

```cmd
robocopy C:\Source D:\Destination *.* /E
```

## Parameters

| Parameter | Description |
| --- | --- |
| `/S` | Copy subdirectories, but skip empty ones |
| `/E` | Copy subdirectories, **including** empty ones |
| `/MIR` | Mirror a directory tree (equivalent to `/E` plus `/PURGE`) — deletes extra dest files |
| `/PURGE` | Delete destination files/dirs that no longer exist in the source |
| `/COPYALL` | Copy all file info (equivalent to `/COPY:DATSOU` — Data, Attributes, Timestamps, ACLs, Owner, aUditing) |
| `/COPY:<flags>` | Choose what to copy: `D`ata, `A`ttributes, `T`imestamps, `S`ecurity(ACLs), `O`wner, a`U`diting |
| `/Z` | Copy in restartable mode (resume after interruption) |
| `/B` | Copy in backup mode (uses backup privilege to bypass ACLs) |
| `/R:<n>` | Number of retries on failed copies (default is 1,000,000) |
| `/W:<n>` | Wait time in seconds between retries (default is 30) |
| `/MT[:n]` | Multithreaded copy with n threads (default 8, range 1–128) |
| `/XO` | Exclude older files (skip source files older than destination) |
| `/XN` | Exclude newer files |
| `/XC` | Exclude changed files |
| `/XD <dirs>` | Exclude directories matching the given names/paths |
| `/XF <files>` | Exclude files matching the given names/wildcards |
| `/LOG:<file>` | Write output to a log file (overwrite) |
| `/LOG+:<file>` | Write output to a log file (append) |
| `/TEE` | Output to console **and** the log file |
| `/NP` | No progress — don't display percentage copied |
| `/L` | List only — show what *would* be copied without copying anything |
| `/FP` | Include full pathnames of files in the output |
| `/ETA` | Show estimated time of arrival for copied files |

## Examples

Copy a directory tree including empty folders:

```cmd
robocopy C:\Data D:\Backup /E
```

Preview a mirror without changing anything (always do this first):

```cmd
robocopy C:\Data D:\Backup /MIR /L
```

Mirror a folder (exact replica — deletes extras in destination):

```cmd
robocopy C:\Data D:\Backup /MIR
```

Copy everything including ACLs, owner, and auditing info:

```cmd
robocopy C:\Share E:\ShareCopy /E /COPYALL
```

Robust large-transfer profile — restartable, multithreaded, limited retries, logged:

```cmd
robocopy \\FS01\Profiles E:\Profiles /E /Z /MT:16 /R:2 /W:5 /LOG:C:\Logs\profiles.log /TEE
```

Copy only `.js` files from a website folder and its subfolders:

```cmd
robocopy C:\Website C:\JS *.js /S
```

Copy multiple file types:

```cmd
robocopy C:\Data D:\Backup *.txt *.log *.csv /S
```

Copy while excluding specific folders and files:

```cmd
robocopy C:\Project D:\ProjectBackup /E /XD node_modules .git /XF *.tmp *.bak
```

Copy only files that are newer in the source (skip older):

```cmd
robocopy C:\Data D:\Backup /E /XO
```

## Enterprise Usage

- **File-server migrations** — `/E /COPYALL /Z /MT` moves shares between servers while preserving NTFS permissions, ownership, and auditing settings.
- **Scheduled backups** — a Task Scheduler job running `robocopy ... /MIR /LOG+:...` keeps a backup target in sync and appends to a rolling log.
- **Roaming/redirected data** — resilient, restartable copies of user profiles and redirected folders over the network.
- **DFS replication seeding** — pre-seed a replica with `/E /COPYALL` so DFS Replication only has to reconcile hashes.

> [!TIP]
> ROBOCOPY exit codes are a **bitmap**, not a simple success/failure. Any value **less than 8** indicates success (files may have been copied, are extra, or mismatched); **8 or higher** indicates at least one failure. In scripts, test `if %ERRORLEVEL% GEQ 8 (...)` rather than `if %ERRORLEVEL% NEQ 0`.

| Exit code (bit) | Meaning |
| --- | --- |
| `0` | No files copied; source and destination already in sync |
| `1` | One or more files copied successfully |
| `2` | Extra files or directories detected in the destination |
| `4` | Mismatched files or directories detected |
| `8` | Some files or directories could not be copied (copy errors) |
| `16` | Serious error — robocopy did not copy any files |

Codes combine additively (e.g. `3` = `1` + `2`: files copied *and* extras present).

## Security Considerations

- `/COPYALL` and `/COPY:S` replicate **NTFS ACLs and ownership** — copying sensitive data to a less-restricted destination can inadvertently widen access; verify destination inheritance.
- `/B` (backup mode) uses the **SeBackupPrivilege** to read files the caller normally cannot access. This is a legitimate admin/backup capability but is also abused to exfiltrate protected files (e.g. copying locked registry hives or profile data) — monitor its use.
- Mirror operations (`/MIR`, `/PURGE`) are destructive; restrict who can run backup/sync scripts and log their output.
- Copying over UNC paths sends data across the network — ensure SMB signing/encryption is in place for sensitive transfers.

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| Access denied on some files | Files locked or ACLs deny read | Run elevated; consider `/B` backup mode with appropriate privilege |
| `/MIR` deleted needed files | Wrong destination or intended one-way copy | Restore from backup; use `/E` (no purge) or preview with `/L` next time |
| Copy is very slow | Single-threaded default over high-latency link | Add `/MT:16` and `/Z` for restartable multithreaded copy |
| Long path errors | Paths exceed legacy limits | ROBOCOPY supports long paths natively; ensure destination filesystem does too |
| Script always reports "error" | Testing exit code with `NEQ 0` | ROBOCOPY returns 1 on success — test `GEQ 8` for real failures |

## References

- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy>
- <https://learn.microsoft.com/en-us/troubleshoot/windows-server/backup-and-storage/robocopy-exit-codes>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [XCOPY-Command](XCOPY-Command.md) — related note — the older directory-copy tool ROBOCOPY supersedes
- [Windows-Basic-Commands](Windows-Basic-Commands.md) — related note — core file and directory commands including `copy`
- [File-System](../File-Services-and-DFS/File-System.md) — related note — NTFS attributes and permissions ROBOCOPY can preserve
