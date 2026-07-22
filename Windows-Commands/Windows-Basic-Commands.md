# Windows Basic Commands

Windows ships with a large collection of built-in command-line utilities for system administration, troubleshooting, automation, networking, file management, and security. These are the everyday commands run from **Command Prompt (`cmd.exe`)**, Windows Terminal, or batch (`.bat`) scripts, and they double as the living-off-the-land toolset an attacker uses post-compromise.

## Overview

Everything in Windows can ultimately be driven from a shell, and the built-in commands below are the foundation of that surface. They cover system inventory, the filesystem, environment variables, networking, and NTFS permissions. For the shell itself and how CMD differs from PowerShell see [Windows-Shell](Windows-Shell.md); for registry editing from the command line see [Windows-Registry](Windows-Registry.md); and for scripting these commands together see [Windows-Batch-Scripting](Windows-Batch-Scripting.md). On an engagement the same built-ins become **living-off-the-land binaries (LOLBins)** — offensively-tuned one-liners live in [PowerShell-Commands-for-Penetration-Testing](PowerShell-Commands-for-Penetration-Testing.md).

> [!NOTE]
> **Elevation**
> Some commands require an **elevated (Run as Administrator)** Command Prompt. Administrative operations — most ownership, ACL, boot, and disk changes — fail with "Access is denied" from a non-elevated shell.

### Prerequisites

- Windows 10, Windows 11, or Windows Server
- Command Prompt (`cmd.exe`) or Windows Terminal
- Administrator privileges for administrative commands

## System Information

### `systeminfo`

Displays detailed information about the operating system, hardware, installed updates, BIOS, memory, and network configuration.

```cmd
systeminfo
```

Example output fields:

| Property | Description |
|-----------|-------------|
| Host Name | Computer name |
| OS Name | Windows edition |
| Version | Windows version |
| Build | Operating system build |
| System Type | x64 or x86 |

### `ver`

Displays the Windows version.

```cmd
ver
```

## User Information

### `whoami`

Displays the currently logged-in user.

```cmd
whoami
```

Display all security information (groups, privileges, SID):

```cmd
whoami /all
```

## Working with Directories

Display the current directory:

```cmd
cd
```

or

```cmd
echo %CD%
```

Change directory:

```cmd
cd C:\Users
```

Move to the Windows directory using an environment variable:

```cmd
cd %SystemRoot%
```

## Environment Variables

Display every environment variable:

```cmd
set
```

Display the executable search path:

```cmd
echo %PATH%
```

Display the current username:

```cmd
echo %USERNAME%
```

Create an environment variable:

```cmd
set DEMO=Armour
```

Read the variable:

```cmd
echo %DEMO%
```

## Displaying Files and Directories

### `dir`

List files and folders.

```cmd
dir
```

Display help:

```cmd
dir /?
```

Display short (8.3) filenames:

```cmd
dir /x
```

List recursively:

```cmd
dir /s
```

Bare format:

```cmd
dir /b
```

Recursive bare format:

```cmd
dir /s /b
```

Hidden and system files:

```cmd
dir /A:SH /S /B
```

> [!TIP]
> **Finding hidden data**
> `dir /A:SH /S /B` and `dir /x` (8.3 short names) are staple discovery commands during post-exploitation — hidden/system files and short-name path tricks are frequently used both to stash payloads and to locate them. See [8.3-Filename-(Short-File-Name)](8.3-Filename-(Short-File-Name).md).

## Creating Directories

Create a directory:

```cmd
md NewFolder
```

or

```cmd
mkdir NewFolder
```

Create multiple directories:

```cmd
md Folder1 Folder2 Folder3
```

Create nested directories:

```cmd
md Parent\Child\GrandChild
```

## Removing Directories

Delete an empty directory:

```cmd
rd FolderName
```

Delete recursively (no prompt):

```cmd
rd FolderName /S /Q
```

## Renaming Files and Folders

```cmd
ren oldname newname
```

or

```cmd
rename oldname newname
```

## Viewing and Writing Files

Display a text file:

```cmd
type file.txt
```

Display text:

```cmd
echo Hello World
```

Write text to a file (overwrite):

```cmd
echo Hello > file.txt
```

Append text:

```cmd
echo Hello >> file.txt
```

## Date and Time

Display or change the date:

```cmd
date
```

Display or change the time:

```cmd
time
```

## Network Information

Display MAC addresses:

```cmd
getmac
```

Display IP configuration:

```cmd
ipconfig
```

Detailed IP information (DNS, DHCP, adapters):

```cmd
ipconfig /all
```

## Command Prompt Control

Change the window title:

```cmd
title Windows Administration
```

Pause execution:

```cmd
pause
```

Change the command prompt string:

```cmd
prompt MyPrompt$
```

Exit Command Prompt:

```cmd
exit
```

## Shutdown and Restart

Restart immediately, forcing running apps to close:

```cmd
shutdown /r /t 0 /f
```

Restart after one minute:

```cmd
shutdown /r /t 60
```

Shutdown after one minute:

```cmd
shutdown /s /t 60
```

Abort a pending shutdown:

```cmd
shutdown /a
```

## Power Management

Disable hibernation:

```cmd
powercfg /h off
```

Generate a battery report:

```cmd
powercfg /batteryreport
```

## File Copying

### `copy`

Display help:

```cmd
copy /?
```

Copy a file:

```cmd
copy source.txt destination.txt
```

Copy to another drive:

```cmd
copy C:\Data\file.txt D:\Backup\
```

### `xcopy`

Copy directories recursively. See [XCOPY-Command](XCOPY-Command.md).

```cmd
xcopy C:\Data D:\Backup /S /Y
```

Copy only HTML files:

```cmd
xcopy C:\inetpub\*.htm C:\HTML /S /Y
```

### `robocopy`

Robust, retryable directory copy — preferred over `xcopy` for bulk transfers. See [ROBOCOPY-Command](ROBOCOPY-Command.md).

```cmd
robocopy /?
```

Copy directories:

```cmd
robocopy C:\Data D:\Backup /S
```

Copy only JavaScript files:

```cmd
robocopy C:\Website C:\JS *.js /S
```

Copy multiple file types:

```cmd
robocopy C:\Data D:\Backup *.txt *.log *.csv /S
```

## File Attributes

### `attrib`

See [ATTRIB-Command](ATTRIB-Command.md) for the full reference.

Display attributes:

```cmd
attrib file.txt
```

Display help:

```cmd
attrib /?
```

Hide a file:

```cmd
attrib +H file.txt
```

Remove hidden attribute:

```cmd
attrib -H file.txt
```

Make a file read-only:

```cmd
attrib +R file.txt
```

Remove read-only:

```cmd
attrib -R file.txt
```

Apply hidden + system recursively:

```cmd
attrib +H +S C:\Data\* /S /D
```

## Ownership

### `takeown`

Take ownership of files and directories — commonly a precursor to re-permissioning system-protected paths. See [TAKEOWN-Command](../File-Services-and-DFS/TAKEOWN-Command.md).

Display help:

```cmd
takeown /?
```

Take ownership of a file:

```cmd
takeown /F file.txt
```

Take ownership recursively:

```cmd
takeown /F C:\Data /R /D Y
```

## NTFS Permissions

### `icacls`

The current tool for viewing and modifying NTFS ACLs. See [ICACLS-Command](../File-Services-and-DFS/ICACLS-Command.md).

Display current permissions:

```cmd
icacls file.txt
```

Grant Full Control:

```cmd
icacls file.txt /grant User:F
```

Grant Modify:

```cmd
icacls file.txt /grant User:M
```

Grant Read:

```cmd
icacls file.txt /grant User:R
```

Remove permissions:

```cmd
icacls file.txt /remove User
```

Reset permissions to inherited defaults:

```cmd
icacls file.txt /reset
```

Enable inheritance:

```cmd
icacls file.txt /inheritance:e
```

Disable inheritance:

```cmd
icacls file.txt /inheritance:d
```

### `cacls` (legacy)

> [!WARNING]
> **Deprecated**
> `CACLS` is deprecated. Use `ICACLS` instead — `cacls` cannot express all modern ACL constructs. See [CACLS-Command](../File-Services-and-DFS/CACLS-Command.md).

Display permissions:

```cmd
cacls file.txt
```

Grant Full Control (edit mode):

```cmd
cacls file.txt /E /G User:F
```

## Reference Tables

### Common Environment Variables

| Variable | Description |
|-----------|-------------|
| `%SystemRoot%` | Windows installation directory |
| `%USERNAME%` | Current username |
| `%USERPROFILE%` | User profile directory |
| `%TEMP%` | Temporary directory |
| `%PATH%` | Executable search path |
| `%CD%` | Current directory |

### Common Permission Codes

| Code | Description |
|------|-------------|
| `F` | Full Control |
| `M` | Modify |
| `RX` | Read & Execute |
| `R` | Read |
| `W` | Write |

## Security Considerations

> [!WARNING]
> **These built-ins are dual-use**
> Every command here is also a **living-off-the-land binary (LOLBin)**. Attackers favor `systeminfo`, `whoami /all`, `ipconfig /all`, `dir /s`, `net`, and `icacls` for enumeration precisely because they are signed, present on every host, and rarely alerted on. Defensively, baseline and log their use; a burst of enumeration commands under one process is a common early post-exploitation signal.

- Avoid granting `Everyone:F` with `icacls` unless absolutely necessary — over-broad ACLs enable privilege escalation via writable files and services.
- Review existing ACLs (`icacls <path>`) before modifying permissions, and be cautious taking ownership of system files.
- Weak permissions on files or directories in the `%PATH%` search order can be abused for DLL/binary hijacking — see [Non-Administrator-User-Write-Permission-Locations-in-Windows](Non-Administrator-User-Write-Permission-Locations-in-Windows.md).
- Test permission and ownership changes in a non-production environment before deployment.

## Best Practices

- Run administrative commands from an elevated Command Prompt (UAC).
- Prefer `ICACLS` over the deprecated `CACLS` for NTFS permission management.
- Use `ROBOCOPY` (with logging and retry limits) instead of `XCOPY` for large file transfers.
- Verify commands before running destructive operations such as `rd /S /Q`, `shutdown`, or `icacls /reset`.
- Use environment variables (`%SystemRoot%`, `%USERPROFILE%`) instead of hardcoded paths.

## Troubleshooting

| Symptom | Likely cause & fix |
|---------|--------------------|
| "Access is denied" running a command | Shell is not elevated — reopen CMD as Administrator (UAC) |
| `icacls` change reports success but is ignored | Inheritance is re-applying an ACE — set `/inheritance:d` before granting/denying |
| `takeown` succeeds but you still cannot edit the file | Ownership is not permission — follow with `icacls <file> /grant User:F` |
| `xcopy` skips subfolders | Missing `/S` (or `/E` to include empty dirs) |
| `%VAR%` prints literally instead of its value | Variable is unset or you are in PowerShell — CMD uses `%VAR%`, PowerShell uses `$env:VAR` |

## References

- [Windows commands reference (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)
- [icacls (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls)
- [robocopy (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy)
- [LOLBAS project — living-off-the-land binaries](https://lolbas-project.github.io/)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows-Shell](Windows-Shell.md) — related note (CMD vs PowerShell, how the shell works)
- [Windows-Registry](Windows-Registry.md) — related note (registry structure and CLI editing)
- [Windows-Batch-Scripting](Windows-Batch-Scripting.md) — related note (scripting these commands together)
- [PowerShell-Commands-for-Penetration-Testing](PowerShell-Commands-for-Penetration-Testing.md) — related note (offensive one-liners)
- [ATTRIB-Command](ATTRIB-Command.md) — related note (`attrib` file attributes)
- [TAKEOWN-Command](../File-Services-and-DFS/TAKEOWN-Command.md) — related note (`takeown` ownership)
- [ICACLS-Command](../File-Services-and-DFS/ICACLS-Command.md) — related note (`icacls` NTFS permissions)
- [CACLS-Command](../File-Services-and-DFS/CACLS-Command.md) — related note (legacy ACL tool)
- [ROBOCOPY-Command](ROBOCOPY-Command.md) — related note (robust file copy)
- [XCOPY-Command](XCOPY-Command.md) — related note (`xcopy` file copy)
- [Windows File System](../File-Services-and-DFS/File-System.md) — related note (NTFS layout and ACLs)
