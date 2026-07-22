# BCDEDIT Command

`bcdedit` is the command-line tool for viewing and editing the **Boot Configuration Data (BCD)** store on Windows Vista and later. The BCD store replaced the legacy `boot.ini` file and describes boot applications and boot-loader settings that the Windows Boot Manager reads at startup.

## Overview

The BCD store is a firmware-independent database of boot-time configuration. `bcdedit` lets administrators enumerate boot entries, change the default operating system, set the boot menu timeout, enable diagnostic/recovery modes (such as Safe Mode via the boot loader), and create or copy boot entries.

`bcdedit` must be run from an **elevated** Command Prompt (Run as administrator). Most edits target one of three well-known identifiers:

| Identifier | Refers to |
| --- | --- |
| `{bootmgr}` | The Windows Boot Manager itself |
| `{current}` | The currently running OS boot entry |
| `{default}` | The entry booted by default when the timeout elapses |

> [!WARNING]
> An incorrect edit to the BCD store can render Windows **unbootable**. Always export a backup with `bcdedit /export` before making changes, and test recovery media is available.

## Syntax

```cmd
bcdedit /command [arguments...]
```

```cmd
bcdedit /?
```

## Parameters

| Parameter | Description |
| --- | --- |
| `/enum` | List entries in the store (e.g. `/enum all`, `/enum active`, `/enum osloader`) |
| `/v` | Verbose — show full GUIDs instead of friendly identifiers |
| `/set <id> <option> <value>` | Set a boot option value on an entry |
| `/deletevalue <id> <option>` | Remove a boot option from an entry |
| `/copy <id> /d "<desc>"` | Copy an existing entry to a new one with a description |
| `/create <id> /d "<desc>"` | Create a new entry |
| `/delete <id>` | Delete an entry from the store |
| `/default <id>` | Set the default boot entry |
| `/displayorder <id> [...]` | Set the order entries appear in the boot menu |
| `/bootsequence <id> [...]` | Set a one-time boot sequence for the next restart |
| `/timeout <seconds>` | Set the boot menu timeout in seconds |
| `/export <file>` | Back up the system BCD store to a file |
| `/import <file>` | Restore the system BCD store from a backup file |
| `/store <file>` | Operate on a specified BCD store file instead of the system store |

## Examples

Display the entire boot configuration:

```cmd
bcdedit /enum all
```

Show verbose output with full GUIDs:

```cmd
bcdedit /enum /v
```

Back up the current BCD store before editing:

```cmd
bcdedit /export C:\BCD_Backup
```

Restore from a backup:

```cmd
bcdedit /import C:\BCD_Backup
```

Set the boot menu timeout to 10 seconds:

```cmd
bcdedit /timeout 10
```

Change the description of the current boot entry:

```cmd
bcdedit /set {current} description "Windows 11 - Production"
```

Set the default OS to the current entry:

```cmd
bcdedit /default {current}
```

Enable minimal Safe Mode at next boot (boot-loader driven):

```cmd
bcdedit /set {default} safeboot minimal
```

Enable Safe Mode with Networking:

```cmd
bcdedit /set {default} safeboot network
```

Remove the Safe Mode setting to return to a normal boot:

```cmd
bcdedit /deletevalue {default} safeboot
```

Copy an entry to create a second, separately configurable boot option:

```cmd
bcdedit /copy {current} /d "Windows 11 - Debug"
```

Enable kernel debugging on an entry (identifier taken from the `/copy` output):

```cmd
bcdedit /debug {default} on
```

Disable the early-launch anti-malware and driver signature checks are separate options; to force a one-time boot to a specific entry for the next restart only:

```cmd
bcdedit /bootsequence {default}
```

Enable boot logging and the legacy F8 advanced boot menu policy:

```cmd
bcdedit /set {bootmgr} displaybootmenu yes
bcdedit /set {default} bootlog yes
```

Reduce the graphics mode / disable the boot-time graphics to expose text diagnostics:

```cmd
bcdedit /set {default} bootux disabled  # untested
```

## Enterprise Usage

- **Dual-boot and staging hosts** — administrators use `/copy`, `/default`, and `/displayorder` to manage multiple OS entries on lab and imaging machines.
- **Standardized timeouts** — set a consistent `/timeout` across a fleet so unattended reboots proceed predictably.
- **Recovery preparation** — `bcdedit /enum` confirms that the Windows Recovery Environment (`{globalsettings}` / recovery sequence) is intact after imaging.
- **Deployment tooling** — imaging and provisioning pipelines (MDT, `bcdboot`, DISM) generate BCD entries; `bcdedit` is used to audit and correct the result.
- Because BCD changes are machine-scoped and require elevation, they are typically applied via a startup script, task sequence, or configuration-management run rather than per-user GPO.

## Security Considerations

- **Integrity of the boot chain** — modifying BCD options such as `nointegritychecks`, `testsigning`, or disabling boot-time protections weakens Secure Boot and driver-signature enforcement. Attackers with local admin may abuse these to load unsigned/malicious drivers (bootkit staging).
- Enabling `safeboot` can be abused to reboot a host into an environment where security agents (EDR/AV) do not start; monitor for unexpected `safeboot` values.
- BCD edits require administrative privileges, so any successful modification implies the actor already holds high privilege — treat unexpected changes as a post-compromise indicator.
- Recording and alerting on `bcdedit` execution and BCD store writes is a useful detection for tamper attempts against boot-time defenses.

> [!IMPORTANT]
> On systems with **BitLocker**, altering the boot configuration can change the platform measurements and trigger the BitLocker recovery key prompt on the next boot. Have the recovery key on hand before editing BCD on an encrypted host.

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| `The boot configuration data store could not be opened. Access is denied.` | Command Prompt not elevated | Re-run in an Administrator Command Prompt |
| System fails to boot after an edit | Bad option or wrong identifier applied | Boot recovery media and run `bcdedit /import` from the backup, or `bootrec /rebuildbcd` |
| BitLocker recovery prompt appears after editing | Boot measurements changed | Enter the recovery key, then optionally suspend BitLocker before further edits |
| Stuck in Safe Mode loop | `safeboot` value left set | Run `bcdedit /deletevalue {default} safeboot` and reboot |
| Changes appear ignored | Edited the wrong entry (default vs current) | Use `bcdedit /enum /v` to confirm the identifier before editing |

## References

- <https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/bcdedit-command-line-options>
- <https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/bcdedit--set>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows-Advanced-Boot-Options](Windows-Advanced-Boot-Options.md) — related note — Safe Mode and recovery boot options BCDEDIT controls
- [DISKPART-Command](DISKPART-Command.md) — related note — partition/volume management that pairs with boot repair
