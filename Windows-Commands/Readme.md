# Windows Commands

The command-line, shell, scripting, and registry surface an administrator — and an attacker — uses to drive Windows.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Everything in Windows can ultimately be driven from a shell. This module is the reference for that surface: core CMD commands and the shell itself, the registry, the `net` / `sc` / `wmic` / `netsh` utility families, disk and boot tooling (`diskpart`, `bcdedit`), file-copy commands (`robocopy`, `xcopy`, `attrib`), batch scripting, and the offensive-relevant details — writable non-admin locations, 8.3 short names, PowerShell for pentesting, and lab-hardening utilities. It doubles as an administration reference and an offensive cheat-sheet.

## Learning Objectives

By the end of this module you will be able to:

- Drive Windows from CMD/PowerShell and edit the registry from the command line
- Use the `net`, `sc`, `wmic`, and `netsh` utility families to manage users, services, WMI, and networking
- Apply file, disk, and boot commands (`robocopy`/`xcopy`/`attrib`/`diskpart`/`bcdedit`) and recognize offensively-relevant command behavior

## Topics Covered

This module contains **23 notes**.

| Note | Topic |
| --- | --- |
| [Windows-Basic-Commands](Windows-Basic-Commands.md) | Everyday CMD commands |
| [Windows-Shell](Windows-Shell.md) | CMD vs PowerShell, how the shell works |
| [Windows-Registry](Windows-Registry.md) | Registry structure and CLI editing |
| [REG-Command](REG-Command.md) | `reg` — registry from the command line |
| [PowerShell-Commands-for-Penetration-Testing](PowerShell-Commands-for-Penetration-Testing.md) | Offensive PowerShell one-liners |
| [Net-Services-Suite](Net-Services-Suite.md) | The `net` command family |
| [Service-Controller-Utility-Commands](Service-Controller-Utility-Commands.md) | `sc` service management |
| [WMIC-Commands](WMIC-Commands.md) | WMI from the CLI |
| [NETSH-Command](NETSH-Command.md) | `netsh` networking configuration |
| [Network-Enumeration](Network-Enumeration.md) | Enumerating hosts, shares, sessions |
| [Windows-Firewall-and-AV-Commands](Windows-Firewall-and-AV-Commands.md) | Firewall/AV from the command line |
| [DISKPART-Command](DISKPART-Command.md) | `diskpart` disk management |
| [BCDEDIT-Command](BCDEDIT-Command.md) | `bcdedit` boot configuration |
| [ROBOCOPY-Command](ROBOCOPY-Command.md) | `robocopy` robust file copy |
| [XCOPY-Command](XCOPY-Command.md) | `xcopy` file copy |
| [ATTRIB-Command](ATTRIB-Command.md) | `attrib` file attributes |
| [Windows-Batch-Scripting](Windows-Batch-Scripting.md) | Batch (`.bat`/`.cmd`) scripting |
| [Windows-Advanced-Boot-Options](Windows-Advanced-Boot-Options.md) | Advanced boot / recovery options |
| [8.3-Filename-(Short-File-Name)](8.3-Filename-(Short-File-Name).md) | 8.3 short filenames |
| [Non-Administrator-User-Write-Permission-Locations-in-Windows](Non-Administrator-User-Write-Permission-Locations-in-Windows.md) | Writable paths for non-admins |
| [rlwrap](rlwrap.md) | Readline/history for raw shells |
| [Win11Debloat](Win11Debloat.md) | Stripping unwanted Windows components |
| [Windows-Defender-Remover](Windows-Defender-Remover.md) | Disabling Defender in a lab |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, work through the utility families on a lab VM: enumerate with `net`/`wmic`, script a task in batch, mirror a folder with `robocopy`, and inspect boot entries with `bcdedit /enum`.

## Best Practices

- Prefer PowerShell for new automation, but know the CMD/`net`/`wmic` equivalents for constrained shells
- Use `robocopy` (with logging and retry limits) over `xcopy` for reliable bulk copies
- Test destructive commands (`diskpart`, `bcdedit`) on snapshots — they can render a system unbootable

## Security Considerations

> [!WARNING]
> **The command line is dual-use**
> The same built-ins that administer Windows are the living-off-the-land binaries attackers rely on to avoid dropping tools.

- `wmic`, `net`, `sc`, `reg`, and `netsh` are common LOLBins — log and baseline their use
- Writable non-admin locations and 8.3 names enable privilege escalation and detection evasion — know them for both offense and defense
- Disabling Defender/hardening is a lab convenience only; never do it on anything reachable from a real network

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| "Access is denied" running a command | Not in an elevated shell — reopen CMD/PowerShell as Administrator (UAC) |
| Batch script behaves differently when double-clicked vs run in shell | Working-directory or environment difference — set an explicit `cd`/paths in the script |

## References

- [Windows commands reference (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)
- [LOLBAS project — living-off-the-land binaries](https://lolbas-project.github.io/)
- [PowerShell documentation](https://learn.microsoft.com/en-us/powershell/)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows PowerShell](../Windows-PowerShell/Readme.md) — related module
- [Windows Operating System Administration](../Windows-Operating-System-Administration/Readme.md) — related module
