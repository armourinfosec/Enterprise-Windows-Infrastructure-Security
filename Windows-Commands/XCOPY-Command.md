# XCOPY Command

`xcopy` is a legacy Windows command-line utility for copying files and **directory trees**, with more capability than the basic `copy` command (recursion, attribute preservation, date filtering, and exclusion lists). It has largely been superseded by `robocopy`, but remains present on all modern Windows versions and is still common in older scripts.

## Overview

Where `copy` handles single files in one directory, `xcopy` can walk subdirectories, recreate folder structure at the destination, copy hidden/system files, and preserve attributes. It is simpler than `robocopy` but lacks restartable transfers, multithreading, and precise NTFS-ACL control.

```cmd
xcopy <source> [destination] [options]
```

> [!TIP]
> For new scripts, prefer [ROBOCOPY](ROBOCOPY-Command.md). It is more robust (restartable, multithreaded, retries, mirror mode, full ACL/owner copy) and is Microsoft's recommended tool for directory replication. Reach for `xcopy` mainly to read or maintain existing scripts.

## Syntax

```cmd
xcopy /?
```

```cmd
xcopy C:\Source D:\Destination /S /E /H /Y
```

## Parameters

| Parameter | Description |
| --- | --- |
| `/S` | Copy directories and subdirectories, except empty ones |
| `/E` | Copy directories and subdirectories, **including** empty ones |
| `/H` | Copy hidden and system files as well |
| `/K` | Copy attributes (by default xcopy resets read-only attributes) |
| `/O` | Copy file ownership and discretionary ACL (DACL) information |
| `/X` | Copy file audit settings and system ACL (SACL) — implies `/O` |
| `/Y` | Suppress prompting to confirm overwriting an existing file |
| `/-Y` | Force the overwrite confirmation prompt |
| `/D[:m-d-y]` | Copy only files changed on or after the date; with no date, copy only files newer than the destination |
| `/EXCLUDE:file1[+file2]` | Skip files whose paths match strings listed in the given file(s) |
| `/I` | If destination doesn't exist and copying more than one file, assume destination is a directory |
| `/C` | Continue copying even if errors occur |
| `/Q` | Quiet — do not display file names while copying |
| `/F` | Display full source and destination file names while copying |
| `/R` | Overwrite read-only files at the destination |
| `/T` | Copy the directory structure only, without files |
| `/Z` | Copy over a network in restartable mode |

## Examples

Copy a directory tree, including empty folders:

```cmd
xcopy C:\Data D:\Backup /S /E /Y
```

Copy everything including hidden and system files:

```cmd
xcopy C:\Data D:\Backup /S /E /H /Y
```

Copy only HTML files from a folder and its subfolders:

```cmd
xcopy C:\inetpub\*.htm C:\HTML /S /Y
```

Copy files while preserving ownership and ACLs:

```cmd
xcopy C:\Share D:\ShareCopy /S /E /O /X /Y
```

Copy only files newer than those already in the destination:

```cmd
xcopy C:\Data D:\Backup /S /D /Y
```

Copy files changed on or after a specific date:

```cmd
xcopy C:\Data D:\Backup /S /D:01-01-2026 /Y
```

Copy the folder structure only (no files):

```cmd
xcopy C:\Project D:\ProjectSkeleton /T /E
```

Exclude paths listed in a text file (one substring per line in `exclude.txt`):

```cmd
xcopy C:\Project D:\ProjectBackup /S /E /EXCLUDE:C:\exclude.txt /Y
```

## Enterprise Usage

- **Legacy logon/startup scripts** — many older environments still deploy files or seed profiles with `xcopy /S /E /Y`; recognize and maintain these safely.
- **Quick ad-hoc tree copies** — for a one-off recursive copy where the robustness of `robocopy` is unnecessary.
- **Image/content staging** — copying web content or application files into place, often with `/EXCLUDE` lists to skip temp/build artifacts.
- New automation should migrate to `robocopy` for logging, retries, and reliable exit codes; when touching an `xcopy` script, consider converting it.

## Security Considerations

- `/O` and `/X` replicate **ownership and ACLs (DACL/SACL)** — copying to a destination with weaker inheritance can unintentionally expose sensitive files; verify effective permissions afterward.
- By default `xcopy` **drops the read-only attribute** on copies (use `/K` to preserve attributes) — a subtle way file protections can be lost during a copy.
- `/H` pulls in hidden and system files, which can include sensitive or unexpected data — be deliberate when mirroring a tree wholesale.
- Unlike `robocopy`, `xcopy` has no restartable-plus-retry-plus-log profile, so partial/failed copies in scripts can go unnoticed — check `%ERRORLEVEL%` (0 = success, non-zero = error).

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| Prompts "Does destination specify a file name or directory name?" | Ambiguous destination for multiple files | Add `/I` to treat the destination as a directory |
| Read-only files not overwritten | Destination has read-only files | Add `/R` to overwrite read-only targets |
| Hidden/system files missing from copy | `/H` not specified | Re-run with `/H` |
| Copied files lost their read-only flag | Default xcopy behavior resets attributes | Use `/K` to keep attributes |
| Network copy fails partway | No restartable/retry logic | Add `/Z`, or switch to `robocopy /Z /R:n /W:n` |
| `Insufficient memory` / path too long | Legacy path-length limits | Use `robocopy`, which supports long paths |

## References

- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/xcopy>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [ROBOCOPY-Command](ROBOCOPY-Command.md) — related note — the modern, robust replacement for XCOPY
- [Windows-Basic-Commands](Windows-Basic-Commands.md) — related note — core file and directory commands including `copy`
