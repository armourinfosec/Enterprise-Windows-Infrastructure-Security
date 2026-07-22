# Backup, Restore and Recovery

Backup, restore, and recovery is the set of Windows Server features and practices — Windows Server Backup (`wbadmin` / the `WindowsServerBackup` PowerShell module), System State backup, Active Directory recovery via DSRM/`ntdsutil`, and Bare-Metal Recovery — that let an administrator get a server, a domain controller, or a whole site back to a working state after hardware failure, corruption, admin error, or ransomware.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course. This module is delivered as a single comprehensive reference below rather than split into separate notes.

## Learning Objectives

By the end of this module you will be able to:

- Plan a backup strategy around RPO/RTO and the 3-2-1 rule, and install/configure Windows Server Backup (`wbadmin` / `WindowsServerBackup` module)
- Choose the correct recovery granularity — file, volume, System State, BMR, or full AD forest recovery — and execute it
- Perform authoritative vs non-authoritative AD restores via DSRM/`ntdsutil`, and troubleshoot boot with WinRE/`bcdedit`/Safe Mode
- Harden backup infrastructure against ransomware and Backup Operators abuse

## Topics Covered

This module is a single in-page reference. Its major sections:

| Section | Topic |
| --- | --- |
| Concept: Why This Matters | RPO/RTO, 3-2-1 rule, restore granularities |
| Windows Server Backup Feature | `wbadmin` and the `WindowsServerBackup` PowerShell module |
| Backup Planning & Scheduling | VSS copy/full modes, scheduling, retention |
| Recovery Strategies Overview | File, volume, application, System State, BMR, AD recovery |
| System State Backup & Restore | DC System State incl. `NTDS.dit`/SYSVOL |
| Active Directory Recovery | Authoritative vs non-authoritative, DSRM, `ntdsutil` |
| Startup / Boot Troubleshooting | WinRE, `bcdedit`, Safe Mode |
| Bare-Metal Recovery (BMR) | Full-server restore to new/dissimilar hardware |
| Disaster Recovery Planning | RPO/RTO/MTD, DR runbook, restore testing |
| Detection / Monitoring & Remediation | Backup-deletion IOCs, Backup Operators hardening |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, on a lab DC: take a System State backup with `wbadmin start systemstatebackup`, delete a test OU, then perform an authoritative restore via DSRM + `ntdsutil` and confirm the OU wins after replication. Separately, rehearse a BMR restore into a fresh VM to prove your backup is actually recoverable.

## Concept: Why This Matters

- Every backup strategy answers two questions: **RPO** (how much data can you afford to lose, i.e. how old can your last good backup be?) and **RTO** (how long can the business tolerate the system being down while you restore?). Backup scheduling and retention are chosen to hit RPO; the restore method (file recovery vs BMR vs full forest recovery) determines RTO.
- The **3-2-1 rule** is the baseline: at least **3** copies of data, on **2** different media types, with **1** copy offsite (or, in modern guidance, air-gapped/immutable). A single on-box backup is not a backup strategy against ransomware, which routinely deletes local shadow copies and backup catalogs before encrypting.
- Windows distinguishes several restore granularities that this note covers in order: **file/folder recovery** → **volume recovery** → **System State recovery** → **Bare-Metal Recovery (BMR)** → **Active Directory forest/domain recovery**. Each is a bigger hammer for a bigger problem.

| Backup type | Contains | Typical use |
|---|---|---|
| File/Folder (item-level) | Selected files/folders | Accidental deletion, single-file recovery |
| Volume | Entire volume, block-level via VSS | Corrupted volume, faster than folder-by-folder restore |
| System State | Boot files, registry, SYSVOL, AD/`NTDS.dit` on DCs, COM+/cert store, IIS metabase, cluster info | Rebuilding a domain controller or fixing OS-level corruption without touching data volumes |
| Bare-Metal Recovery (BMR) | All critical volumes needed to boot (OS + boot + system state) | Full server replacement — restore to new/dissimilar hardware or a VM |
| Full Server | Every volume on the machine | Complete disaster recovery of a single server |

## Windows Server Backup Feature

### Installing Windows Server Backup

Windows Server Backup is a **Feature**, not a Role, and is not installed by default.

```powershell
# Install the feature and its management tools (GUI snap-in + wbadmin + PS module)
Install-WindowsFeature -Name Windows-Server-Backup -IncludeManagementTools  # untested

# Confirm installation state
Get-WindowsFeature -Name Windows-Server-Backup  # untested
```

Via Server Manager: **Add Roles and Features → Features → Windows Server Backup**.

### `wbadmin` — command-line tool

`wbadmin` is the elevated command-prompt tool for Windows Server Backup; it replaced the older `ntbackup`. It must be run from an **elevated** prompt by a member of **Backup Operators** or **Administrators**.

| Subcommand | Purpose |
|---|---|
| `wbadmin enable backup` | Creates/modifies a scheduled daily backup |
| `wbadmin disable backup` | Disables the scheduled daily backup |
| `wbadmin start backup` | Runs a one-time backup |
| `wbadmin start systemstatebackup` | Backs up System State only |
| `wbadmin start systemstaterecovery` | Restores System State from a backup |
| `wbadmin start recovery` | Restores files/folders/volumes/apps from a backup |
| `wbadmin start sysrecovery` | Performs a full Bare-Metal Recovery (used from WinRE) |
| `wbadmin get versions` | Lists available backup versions (time, location, version ID, recovery types) |
| `wbadmin get status` | Reports status of the currently running backup/recovery job |
| `wbadmin get disks` | Lists disk identifiers usable as backup targets |
| `wbadmin restore catalog` | Recovers the backup catalog from a storage location |
| `wbadmin delete catalog` | Deletes the backup catalog on the local computer (destructive) |
| `wbadmin stop job` | Stops the currently running backup/recovery operation |

Reference: [wbadmin (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin)

```cmd
:: One-time backup of E: and D:\mountpoint to F:, including System State,
:: as a VSS copy backup (does not disturb scheduled incremental history)
wbadmin start backup -backupTarget:f: -include:e:,d:\mountpoint -systemstate -vsscopy -quiet
```

```cmd
:: List available backup versions on volume H: (needed to get -version: for a restore)
wbadmin get versions -backupTarget:h:
```

```cmd
:: Check on a currently running backup/recovery job
wbadmin get status
```

Source for the syntax above: [wbadmin start backup](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin-start-backup), [wbadmin get versions](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin-get-versions), [wbadmin get status](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin-get-status).

### `WindowsServerBackup` PowerShell module

The module mirrors `wbadmin` as PowerShell cmdlets built around a `WBPolicy` object that you configure and then apply/run.

| Cmdlet | Purpose |
|---|---|
| `New-WBPolicy` | Creates a new backup policy object (edit mode) |
| `Get-WBPolicy [-Editable]` | Gets the current scheduled backup policy; `-Editable` returns it in edit mode |
| `Add-WBVolume` / `Get-WBVolume` | Adds a volume to a policy |
| `Add-WBFileSpec` / `New-WBFileSpec` | Adds specific files/folders to a policy |
| `Add-WBSystemState` | Adds System State to the policy |
| `Add-WBBareMetalRecovery` | Marks the policy for Bare-Metal Recovery capability |
| `New-WBBackupTarget` / `Add-WBBackupTarget` | Defines and attaches the backup storage location |
| `Set-WBVssBackupOption` | Sets VSS full vs copy backup mode on the policy |
| `Set-WBSchedule` | Sets the daily backup time(s) on a policy |
| `Set-WBPolicy` | Applies an edited policy as the active scheduled-backup policy |
| `Start-WBBackup -Policy $Policy` | Runs a backup once, using a policy (new or existing) |
| `Get-WBJob` | Gets status of the running backup/recovery job |
| `Get-WBSummary` | Gets the history of backup operations |
| `Start-WBSystemStateRecovery` | Starts a System State recovery |
| `Start-WBFileRecovery` / `Start-WBVolumeRecovery` | Restores files or an entire volume |

```powershell
# Build and run a one-time backup: system state + volume F: + BMR capability, to D:
$Policy = New-WBPolicy

$Volume = Get-WBVolume -VolumePath "F:"
Add-WBVolume -Policy $Policy -Volume $Volume

Add-WBSystemState -Policy $Policy
Add-WBBareMetalRecovery -Policy $Policy

$Target = New-WBBackupTarget -VolumePath "D:"
Add-WBBackupTarget -Policy $Policy -Target $Target

Set-WBVssBackupOption -Policy $Policy -VssCopyBackup

Start-WBBackup -Policy $Policy
```

This sequence is adapted directly from the documented `Start-WBBackup` example. Source: [Start-WBBackup](https://learn.microsoft.com/en-us/powershell/module/windowsserverbackup/start-wbbackup) (Microsoft Learn); the full cmdlet index is at [WindowsServerBackup module](https://learn.microsoft.com/en-us/powershell/module/windowsserverbackup/).

```powershell
# Check the running backup, and review history of past runs
Get-WBJob
Get-WBSummary
```

## Backup Planning & Scheduling

### Backup modes

Windows Server Backup uses **VSS** (Volume Shadow Copy Service) rather than the classic full/incremental/differential model:

| Mode | Flag | Behaviour |
|---|---|---|
| VSS Copy backup | `-vssCopy` (default) | Backs up everything, but does **not** update each file's "backed up" history — safe to run alongside a 3rd-party backup product's own incremental chain |
| VSS Full backup | `-vssFull` | Backs up everything **and** updates file history/truncates logs — subsequent scheduled runs then only capture *changed blocks*, giving incremental-like efficiency without needing a separate flag |

Source: [wbadmin start backup — parameters](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin-start-backup).

### Scheduling a recurring backup

```cmd
:: Schedule daily backups at 09:00 and 18:00 of E:, D:\mountpoint to disk "DiskID"
:: (wbadmin get disks gives you the disk identifier)
wbadmin enable backup -addtarget:DiskID -schedule:09:00,18:00 -include:E:,D:\mountpoint -systemstate -vssfull -quiet
```

```powershell
# Same idea via PowerShell: schedule an existing/edited policy for 09:00 and 21:00
$Policy = Get-WBPolicy -Editable
Set-WBSchedule -Policy $Policy -Schedule 09:00,21:00
Set-WBPolicy -Policy $Policy
```

Sources: [wbadmin enable backup](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin-enable-backup), [Set-WBSchedule](https://learn.microsoft.com/en-us/powershell/module/windowsserverbackup/set-wbschedule).

### Retention

- `-addtarget` with a **dedicated disk** lets Windows Server Backup manage retention itself: it reformats the disk on first use and automatically ages out the oldest backups as space fills, per the documented `wbadmin enable backup` behaviour.
- Backups saved to a **remote shared folder** are **overwritten** each time you back up the same computer to the same share — there is no automatic version history there. Microsoft's own guidance is to create per-run subfolders if you need multiple recovery points on a share, at the cost of roughly double the storage.
- There is no separate "retention in days" knob for on-disk Windows Server Backup targets the way there is for many 3rd-party products — retention is a function of disk capacity and rotation, which is why dedicating enough disk (or offloading older backups offsite) is part of backup planning, not an afterthought.

## Recovery Strategies Overview

| Recovery type | Tool | When to use |
|---|---|---|
| File/folder recovery | `wbadmin start recovery`, `Start-WBFileRecovery` | Accidental delete/overwrite of specific files |
| Volume recovery | `wbadmin start recovery`, `Start-WBVolumeRecovery` | Corrupted data volume, faster than per-file restore |
| Application recovery | `wbadmin start recovery -recoveryTarget:app`, `Start-WBApplicationRecovery` | VSS-aware app (e.g. Hyper-V) needs a consistent restore |
| System State recovery | `wbadmin start systemstaterecovery`, `Start-WBSystemStateRecovery` | Registry/boot files/AD corruption without touching data |
| Bare-Metal Recovery | `wbadmin start sysrecovery` from WinRE, or **System Image Recovery** in WinRE UI | Server won't boot, or restoring to new/replacement hardware |
| AD forest/domain recovery | Authoritative/non-authoritative restore + `ntdsutil`, or full [Active-Directory-Domain-Services](../Active-Directory-Domain-Services-AD-DS/Active-Directory-Domain-Services.md) forest recovery | Domain-wide AD corruption, mass object deletion, or forest-wide compromise |

## System State Backup & Restore

System State on a domain controller includes: boot files, the registry, `SYSVOL` (Group Policy + logon scripts), **Active Directory and `NTDS.dit`**, and the certificate store if AD CS is installed. See [SAM-vs-NTDS.dit](../Active-Directory-Domain-Services-AD-DS/SAM-vs-NTDS.dit.md) for how `NTDS.dit` differs from the SAM on a member server.

```cmd
:: Back up System State only, to volume F:
wbadmin start systemstatebackup -backupTarget:f:
```

```cmd
:: Restore System State from the backup identified by wbadmin get versions
wbadmin start systemstaterecovery -version:03/31/2026-09:00
```

Sources: [wbadmin start systemstatebackup](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin-start-systemstatebackup), [wbadmin start systemstaterecovery](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin-start-systemstaterecovery). Note that Windows Server Backup does **not** back up or recover `HKEY_CURRENT_USER` registry hives as part of System State (documented limitation).

```powershell
Start-WBSystemStateRecovery -BackupSet $BackupSet  # untested — verify exact parameter set with Get-WBBackupSet in your build before running
```

On a domain controller, a System State recovery runs in the context of **DSRM** and, unless `-authsysvol` is used, restores AD **non-authoritatively** — see the Active Directory Recovery section below for the authoritative-vs-non-authoritative distinction.

## Active Directory Recovery

### Authoritative vs non-authoritative restore

- **Non-authoritative restore** (the default) restores a DC's AD database to its state at backup time, then lets normal AD replication bring it back up to date from its replication partners. Use this to recover a single failed/corrupted DC when other healthy DCs exist.
- **Authoritative restore** marks specific restored objects (or a subtree) with a higher version number so that, on the next replication cycle, those objects **overwrite** the (newer but wrong) copies on every other DC — instead of being immediately overwritten themselves. Use this when the problem is **bad data that already replicated everywhere** (e.g. an OU deleted domain-wide) and you need the restored copy to win.

### DSRM (Directory Services Restore Mode)

DSRM is a special safe-mode-like boot state for a domain controller in which AD DS is offline and the DC authenticates using the local DSRM Administrator account/password (set at `dcpromo`/`Install-ADDSDomainController` time) instead of a domain account. Authoritative restores with `ntdsutil` must be performed while the DC is booted into DSRM.

```cmd
:: Reboot into DSRM (legacy method: msconfig > Boot > Safe boot > Active Directory repair
:: is the GUI equivalent). Reboot required to take effect, and required again to leave DSRM.
bcdedit /set safeboot dsrepair  # untested
shutdown /r /t 0                # untested
:: after the restore, remove the safeboot flag so the DC boots normally again:
bcdedit /deletevalue safeboot   # untested
```

```cmd
:: Reset/verify the DSRM Administrator password from a live DC (does not require a reboot)
ntdsutil "set dsrm password" "reset password on server null" quit quit  # untested
```

### `ntdsutil` — authoritative restore

Run from an elevated command prompt **while booted into DSRM**:

```cmd
ntdsutil
activate instance ntds
authoritative restore
restore subtree "OU=Sales,DC=corp,DC=local"
quit
quit
```
`# untested — sequence follows documented ntdsutil authoritative-restore syntax; validate DN and instance name in your environment before running against a production DC`

- `restore subtree <DN>` restores and marks authoritative a specific OU/container; `restore database` (whole-database variant) marks the **entire** database authoritative — a much bigger blast radius, normally reserved for full forest recovery scenarios.
- After an authoritative restore, reboot the DC out of DSRM and let it replicate; the version-incremented objects will propagate outward and overwrite peers.

### Tombstone lifetime and USN rollback

- A System State/AD backup is only useful for restore within the domain's **tombstone lifetime** (default 180 days on modern AD, historically 60 days on older forests) — beyond that, deleted-object tombstones needed for replication consistency have been garbage-collected, and Microsoft's guidance is such backups should not be restored.
- Restoring/cloning a DC's `NTDS.dit` without going through a supported backup/VM-snapshot-aware restore path risks a **USN rollback**, where the DC's Update Sequence Numbers regress and it stops replicating correctly with peers that have already seen the "future" USNs — this is why ad hoc VM snapshot rollback of a DC is explicitly unsupported without VM-Generation ID awareness (Hyper-V/VMware handle this on modern platforms, but it's a known DR gotcha worth flagging in a DR runbook).

## Startup / Boot Troubleshooting

### WinRE (Windows Recovery Environment)

WinRE is the WinPE-based recovery environment preloaded on Windows Server 2016+ and Windows 10/11, used for automatic repair, System Image Recovery (BMR), and other troubleshooting tools. It is entered via the **Advanced Startup** menu:

- From the sign-in screen: **Shut down → hold Shift → Restart**.
- **Settings → Update & Security → Recovery → Advanced startup → Restart now**.
- Booting from Windows installation/recovery media and choosing **Repair your computer**.
- WinRE also starts **automatically** after two consecutive failed boot attempts or two consecutive unexpected shutdowns within two minutes of boot completion.

Source: [Windows Recovery Environment (Windows RE) technical reference](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-recovery-environment--windows-re--technical-reference).

```text
Advanced Startup menu options:
Continue → Troubleshoot → Reset this PC / Advanced options
  Advanced options → System Restore, System Image Recovery,
                      Startup Repair, Command Prompt, Startup Settings, UEFI Firmware Settings
```

### `bcdedit`

`bcdedit` edits the Boot Configuration Data store — used to inspect the boot menu, force Safe Mode at next boot, or point at a recovery/BMR target.

| Command | Purpose |
|---|---|
| `bcdedit /enum` | Lists current boot entries |
| `bcdedit /set {current} safeboot minimal` | Forces next boot into Safe Mode |
| `bcdedit /set {current} safeboot network` | Forces next boot into Safe Mode with Networking |
| `bcdedit /set safeboot dsrepair` | Forces next boot into DSRM on a DC |
| `bcdedit /deletevalue safeboot` | Clears the forced safe-boot flag |
| `bcdedit /set {bootmgr} timeout 30` | Sets boot menu timeout in seconds |

`# untested — standard, long-stable bcdedit syntax from general Windows Server administration knowledge; not re-verified against Microsoft Learn in this session`

### Safe Mode

```cmd
:: Force one Safe Mode boot then automatically clear the flag (Windows 8+/2012+)
shutdown /r /o /t 0  # untested — opens the Advanced Startup menu on restart to choose Safe Mode manually
```

The GUI equivalent is `msconfig.exe` → **Boot** tab → **Safe boot**.

## Bare-Metal Recovery (BMR)

BMR restores all critical volumes needed to boot a server (OS volume, boot volume, and System State) as a single operation — typically to replacement or dissimilar hardware, or into a VM. A backup is only BMR-capable if it was taken with `-allCritical` (`wbadmin`) or `Add-WBBareMetalRecovery` (PowerShell), as shown in the Windows Server Backup section above.

To perform BMR:

1. Boot the target machine from Windows Server installation media or a WinRE/recovery USB.
2. Choose **Repair your computer → Troubleshoot → Advanced options → System Image Recovery**.
3. Point it at the backup location (local disk, network share, or removable media) containing an `-allCritical`/BMR-capable backup.
4. Alternatively, from a WinRE/WinPE command prompt: `wbadmin start sysrecovery -version:<id> -backupTarget:<location>` `# untested — sysrecovery is intended to run from WinRE, not a live OS, and wasn't re-verified against Microsoft Learn this session`.

BMR is the fastest path to RTO for "the box is gone" scenarios, but it restores **to the same disk layout/critical-volume set** the backup was taken from — capacity and partition-scheme mismatches on the replacement hardware are the most common real-world BMR failure mode, which is why DR runbooks should note the exact disk layout a BMR backup expects.

## Disaster Recovery Planning

| Term | Meaning |
|---|---|
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss, measured in time — driven by backup *frequency* |
| **RTO** (Recovery Time Objective) | Maximum acceptable downtime — driven by restore *method* (file restore is fast; full forest recovery is slow) |
| **MTD** (Maximum Tolerable Downtime) | The hard business ceiling RTO must stay under |

- Follow the **3-2-1 rule** (3 copies, 2 media types, 1 offsite) and, for ransomware resilience specifically, keep at least one copy **offline or immutable** — attackers routinely run `vssadmin delete shadows /all /quiet` and `wbadmin delete catalog -quiet` against reachable Windows backup infrastructure before detonating ransomware, which defeats any backup that a compromised admin account can reach and delete.
- Maintain a written DR runbook: backup inventory (what's backed up, where, how often), restore procedures per recovery type from this note, named roles/contacts, and DSRM passwords stored securely and **out-of-band** from the domain they protect.
- **Test restores regularly.** A backup that has never been restored is a hypothesis, not a plan — `wbadmin get versions` proving a backup *exists* is not proof it *restores*.
- For AD specifically, maintain **at least two DCs backed up with System State** so authoritative restore always has a healthy replication partner to seed from, and periodically validate that backups are within the tombstone-lifetime window.

## Evidence: What Success Looks Like

- `wbadmin get versions` succeeding lists, per the documented output shape, each backup's time, storage location, a version identifier (used as `-version:` for a restore), and the types of recovery that backup supports — an empty or erroring result means there is currently no usable backup to restore from, which is itself an important thing to notice *before* an incident.
- `Get-WBJob` / `Get-WBSummary` (PowerShell) or `wbadmin get status` (cmd) reporting a completed job with no failed items is the operational confirmation a scheduled or one-time backup actually ran, rather than silently failing (e.g. due to a full target disk or a broken VSS writer).
- A successful System State/BMR restore is confirmed by the server booting normally and, for a DC, `dcdiag` and replication (`repadmin /replsummary`) coming back clean after the post-restore reboot — this note does not fabricate a specific console transcript; validate against your own environment.

## Detection / Monitoring

- Windows Server Backup writes operational events to **Applications and Services Logs → Microsoft-Windows-Backup → Operational** — monitor this channel for failed scheduled backups rather than discovering the gap only when a restore is needed. See [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md) for general Windows Event Log/`Get-WinEvent` usage.
- Alert on execution of `vssadmin delete shadows`, `wmic shadowcopy delete`, `wbadmin delete catalog`, and `wbadmin delete systemstatebackup` — these are near-universal ransomware pre-encryption steps to remove local recovery options, and legitimate use of them by anyone other than backup administration is rare enough to be high-signal. Correlate with process-creation logging (Event ID 4688 / Sysmon Event ID 1) — see [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md).
- On domain controllers, monitor for unexpected DSRM boots and for `ntdsutil` authoritative-restore activity outside of a planned maintenance window; an unplanned authoritative restore is either a real incident response action or a sign an attacker is manipulating AD replication.
- Track **USN rollback** warnings in the Directory Service event log — they indicate a DC was restored/rolled back outside a supported path and needs to be reviewed before it's trusted to replicate further.

## Remediation / Hardening Notes

- Restrict membership of the **Backup Operators** group tightly — it is a de facto privilege-escalation path (a Backup Operator can back up, and therefore read, `NTDS.dit` and the registry SAM/SECURITY hives on any machine they can reach) as well as the group that legitimately needs `wbadmin`/`vssadmin` access.
- Keep at least one backup copy immutable or offline/air-gapped so it can't be deleted by the same compromised credentials that reach production.
- Store DSRM passwords in a password manager/vault, not written down near the DC or reused across DCs; rotate them like any other privileged credential.
- Protect `SYSVOL`/`NTDS.dit` backups with the same access controls as the live directory — a stolen System State backup gives an attacker the same offline hash-extraction opportunities as [SAM-vs-NTDS.dit](../Active-Directory-Domain-Services-AD-DS/SAM-vs-NTDS.dit.md) describes for the live database.
- Validate BMR backups against the actual replacement-hardware/VM disk layout you'd use in a real DR event, not just against the original server.

## References

- [Windows Server Backup (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-server-backup/)
- [`wbadmin` command reference (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin)
- [AD forest recovery guide (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/ad-forest-recovery-guide)
- [MITRE ATT&CK — Inhibit System Recovery (T1490)](https://attack.mitre.org/techniques/T1490/)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Active-Directory-Domain-Services](../Active-Directory-Domain-Services-AD-DS/Active-Directory-Domain-Services.md) — AD DS this note's AD-recovery section restores
- [SAM-vs-NTDS.dit](../Active-Directory-Domain-Services-AD-DS/SAM-vs-NTDS.dit.md) — what's inside the credential store this note backs up/restores
- [RAID-(Redundant-Array-of-Independent-Disks)](../File-Services-and-DFS/RAID-(Redundant-Array-of-Independent-Disks).md) — disk redundancy as a complement (not substitute) for backup
- [File Services and DFS](../File-Services-and-DFS/Readme.md) — disk/volume concepts referenced by backup targets and BMR
- [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md) — event log monitoring for backup jobs and ransomware TTPs
- [Booting-Process](../Fundamental-Of-Operating-System/Booting-Process.md) — boot sequence context for WinRE/Safe Mode/DSRM
