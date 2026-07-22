# Microsoft Windows Activation

Microsoft Windows Activation is the licensing check that confirms a Windows (or Office) installation is genuine and properly licensed. In a throwaway pentest lab, activation matters mainly so evaluation and practice VMs stay usable past their grace period — this note covers both the legitimate mechanisms (evaluation media, `slmgr`, KMS) and the third-party tooling commonly used to activate lab machines.

## Overview

For lab work you rarely need retail licenses. Microsoft ships time-limited **evaluation** builds (see [Windows-Evaluation-Center](Windows-Evaluation-Center.md)) and free **Windows Server** trials that run 180 days and can be re-armed, which is usually enough for a disposable [virtual machine](Virtualization.md). When a longer-lived lab host is wanted, practitioners often reach for community activation scripts. Legitimate activation is driven by the built-in `slmgr.vbs` tool and, in enterprise environments, a **KMS** (Key Management Service) host — the same volume-activation machinery you will meet again on real [Windows editions](../Fundamental-Of-Operating-System/Windows-Operating-System-Editions.md) and domain-joined servers.

> [!WARNING]
> **Licensing and legality**
> Third-party activation scripts and Defender/Update removers are used here purely for **disposable, offline lab VMs**. Do not run them against machines you do not own, production systems, or anything holding real data. Prefer genuine evaluation media whenever it will do the job.

## Activation Options

| Option | Best for | Notes |
| --- | --- | --- |
| Evaluation media | Short-lived DC / client VMs | 180-day Server trial; re-armable. See [Windows-Evaluation-Center](Windows-Evaluation-Center.md) |
| `slmgr` + product key / KMS | Persistent lab hosts you have keys for | Built-in, fully legitimate |
| Microsoft Activation Scripts (MAS) | Long-lived personal lab VMs | Community tooling; lab use only |

## Legitimate Activation with slmgr

`slmgr.vbs` (Software Licensing Management tool) is the built-in Windows utility for viewing and changing activation state. Run these from an elevated **Command Prompt** or **PowerShell**.

Display detailed license status and the activation expiry:

```cmd
slmgr /dlv
slmgr /xpr
```

Install a product key, then activate against Microsoft or a KMS host:

```cmd
slmgr /ipk XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
slmgr /ato
```

Point an edition that uses volume activation at a KMS server:

```cmd
slmgr /skms kms.lab.local:1688
slmgr /ato
```

Reset the activation grace period on an evaluation build (re-arm):

```cmd
slmgr /rearm
```

> [!TIP]
> **Re-arm evaluation VMs instead of reinstalling**
> Evaluation Windows Server builds grant a 180-day grace period that `slmgr /rearm` can reset a limited number of times. For a lab that just needs to keep running, re-arming is faster and cleaner than rebuilding the VM — snapshot first so you can roll back.

## Microsoft Activation Scripts (MAS)

Microsoft Activation Scripts is a widely used open-source project for activating Windows 10/11 and Office. It is convenient for long-lived personal lab VMs but is **community tooling** — treat it as lab-only.

**Repository:** [Microsoft Activation Scripts](https://github.com/massgravel/Microsoft-Activation-Scripts)

### Method 1 — PowerShell one-liner

Open **PowerShell** (Start menu → type "PowerShell"), then run the command for your Windows version.

For Windows 8 / 10 / 11:

```powershell
irm https://get.activated.win | iex
```

If access is blocked by ISP/DNS filtering, resolve over DNS-over-HTTPS:

```powershell
iex (curl.exe -s --doh-url https://1.1.1.1/dns-query https://get.activated.win | Out-String)
```

Older `DownloadString` cradle (Windows 7 and later):

```powershell
iex ((New-Object Net.WebClient).DownloadString('https://get.activated.win'))
```

Then follow the on-screen menu, selecting the highlighted options to activate Windows or Office.

### Method 2 — All-in-One script from the repo

1. Download the repository from GitHub (or clone it).
2. Extract the contents of the ZIP file.
3. Run `MAS_AIO.cmd` as Administrator.
4. Follow the on-screen instructions to activate Windows or Office.

Clone the repository with Git:

```bash
git clone https://github.com/massgravel/Microsoft-Activation-Scripts.git
```

## Genuine Installation Media

Download genuine, unmodified Windows installation media directly from official sources so you start from a clean base image. See also [Custom-build-Windows-11-ISO](Custom-build-Windows-11-ISO.md) for building a customized image on top of genuine media.

- [Genuine Installation Media](https://massgrave.dev/genuine-installation-media)
- [Windows Server Installation Media](https://massgrave.dev/windows_server_links)
- [Windows 11 Installation Media](https://massgrave.dev/windows_11_links)

Open the download page from PowerShell:

```powershell
# Open the Windows 11 media link in the default browser
Start-Process "https://massgrave.dev/windows_11_links"
```

## Windows Defender and Update Removal

These tools disable or remove **Windows Defender** and **Windows Update** so a lab VM stops auto-updating or quarantining offensive tooling. They intentionally weaken the host — only ever run them on isolated, disposable lab machines. See [Windows-Defender-Remover](../Windows-Commands/Windows-Defender-Remover.md) for a dedicated note on this tooling.

### Windows Defender Remover

Removes Windows Defender from the system.

**Repository:** [Windows Defender Remover](https://github.com/jbara2002/windows-defender-remover)

```bash
git clone https://github.com/jbara2002/windows-defender-remover.git
```

Run the removal script (elevated):

```powershell
cd windows-defender-remover
.\Remove-WindowsDefender.ps1   # untested
```

### Windows Update Disabler

Disables Windows Update.

**Repository:** [Windows Update Disabler](https://github.com/tsgrgo/windows-update-disabler)

```bash
git clone https://github.com/tsgrgo/windows-update-disabler.git
```

```powershell
cd windows-update-disabler
.\Disable-WindowsUpdate.ps1   # untested
```

### Windows Update Notification Blocker

Blocks Windows update notifications without fully disabling the update service.

**Repository:** [Windows 10 Update Notification Blocker](https://github.com/Voltstriker/Windows-10-Update-Notification-Blocker)

```bash
git clone https://github.com/Voltstriker/Windows-10-Update-Notification-Blocker.git
```

```powershell
cd Windows-10-Update-Notification-Blocker
.\Block-Update-Notification.ps1   # untested
```

## Security Considerations

> [!WARNING]
> **`irm | iex` is the same pattern attackers use**
> Piping a remote script straight into `iex` (an **execution cradle**) runs whatever the server returns, with no signature or review — the exact download-and-execute technique used by malware droppers and red-team payloads. Only run these cradles on isolated lab VMs, over a channel you trust, and understand that the remote content can change at any time.

- **Supply-chain risk** — third-party activation and remover scripts are unsigned community code; a compromised repo or hijacked domain would run with your privileges. Review the code and pin to a known-good commit for anything you keep.
- **Deliberately weakened host** — removing Defender and disabling Update leaves the VM without AV and unpatched. Never bridge such a machine onto a real network (see [VirtualBox-Network-Modes](VirtualBox-Network-Modes.md) for isolating lab traffic).
- **Defensive relevance** — activation scripts, Defender-removal tooling, and `slmgr` tampering are IOCs a blue team should alert on; recognizing them is useful on both sides of an engagement.
- **Snapshot first** — take a clean [VM snapshot](Virtualization.md) before running any of this so a bad script is a rollback, not a rebuild.

## Best Practices

- Prefer genuine **evaluation media** and `slmgr /rearm` over activation scripts whenever the lab timeframe allows.
- Run activation and remover tooling **only** on isolated, disposable lab VMs — never production or shared machines.
- Read and pin third-party scripts to a specific commit; do not blindly trust `latest`.
- Snapshot every VM in a known-good state before making activation or Defender/Update changes, and roll back afterward.
- Keep a backup of any VM before removing Defender or disabling updates, since these changes are hard to fully reverse.

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Activation fails | No internet route, or AV/firewall blocking — connect the VM, temporarily disable interfering security software, retry |
| `get.activated.win` unreachable | ISP/DNS filtering — use the DNS-over-HTTPS one-liner above |
| PowerShell execution-policy error | Loosen policy for the current process: `Set-ExecutionPolicy Bypass -Scope Process -Force` |
| "Access denied" running a script | Not elevated — re-open PowerShell/Command Prompt **as Administrator** |
| Evaluation build reports expired | Re-arm the grace period with `slmgr /rearm`, then reboot |

## References

- [Microsoft Learn — Slmgr.vbs options for volume activation](https://learn.microsoft.com/windows-server/get-started/activation-slmgr-vbs-options)
- [Microsoft Learn — Volume Activation overview](https://learn.microsoft.com/windows/deployment/volume-activation/volume-activation-windows-10)
- [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub
- [Windows-Evaluation-Center](Windows-Evaluation-Center.md) — legally sourcing time-limited Windows media
- [Custom-build-Windows-11-ISO](Custom-build-Windows-11-ISO.md) — activation of customized builds
- [Windows-Operating-System-Editions](../Fundamental-Of-Operating-System/Windows-Operating-System-Editions.md) — activation and features differ per edition
- [Windows-Defender-Remover](../Windows-Commands/Windows-Defender-Remover.md) — dedicated note on Defender removal tooling
- [Virtualization](Virtualization.md) — snapshots and disposable VMs for lab activation
- [VirtualBox-Network-Modes](VirtualBox-Network-Modes.md) — isolating a weakened lab host from real networks
