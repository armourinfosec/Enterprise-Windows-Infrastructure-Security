# Windows PowerShell

The automation and configuration engine of modern Windows — the administrator's primary tool and the attacker's favourite living-off-the-land shell.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

PowerShell is the object-oriented shell and scripting language that drives Windows and Windows Server administration end to end — from local configuration to fleet-wide remoting. This module builds PowerShell from language fundamentals through remoting, modules, and secure execution, and covers both sides of its dual-use nature: the logging and lockdown features (script-block logging, transcription, Constrained Language Mode, JEA) that defenders rely on, and the offensive tradecraft that makes PowerShell a top living-off-the-land technique.

## Learning Objectives

By the end of this module you will be able to:

- Write PowerShell scripts using cmdlets, the pipeline, objects, and modules
- Administer remote hosts securely with PowerShell Remoting (WinRM/SSH) and PSSessions
- Configure execution policy, script-block logging, transcription, Constrained Language Mode, and JEA — and understand how each is attacked and bypassed

## Topics Covered

This module contains **7 notes**.

| Note | Topic |
| --- | --- |
| [PowerShell-Language-Fundamentals](PowerShell-Language-Fundamentals.md) | Cmdlets, pipeline, objects, variables, control flow |
| [PowerShell-Modules-and-Profiles](PowerShell-Modules-and-Profiles.md) | Importing, authoring, and managing modules |
| [PowerShell-Remoting](PowerShell-Remoting.md) | WinRM/SSH remoting, `Enter-PSSession`, `Invoke-Command` |
| [Execution-Policy-and-Signing](Execution-Policy-and-Signing.md) | Execution policy, code signing, bypasses |
| [PowerShell-Logging](PowerShell-Logging.md) | Script-block logging, module logging, transcription |
| [Constrained-Language-Mode-and-JEA](Constrained-Language-Mode-and-JEA.md) | Constrained Language Mode and JEA for least privilege |
| [Offensive-PowerShell](Offensive-PowerShell.md) | LOLBin usage, download cradles, AMSI context |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection. Planned exercises include enabling and reading script-block logging, restricting a user to a JEA endpoint, and comparing an administrative task done in the GUI vs PowerShell remoting.

## Best Practices

- Prefer PowerShell for repeatable administration; keep scripts in version control and sign them
- Enable script-block logging, module logging, and transcription centrally via GPO — visibility first
- Use Constrained Language Mode and JEA to give operators only the cmdlets their role needs

## Security Considerations

> [!WARNING]
> **PowerShell is dual-use by design**
> The same remoting and scripting power that automates a fleet is exactly what attackers use to run code without dropping tools — log it, don't just block it.

- Execution policy is **not** a security boundary — it is trivially bypassed; rely on WDAC/AppLocker + CLM instead
- Attackers disable/evade AMSI and logging — monitor for logging tampering, not just script content
- PowerShell Remoting (WinRM) is a primary lateral-movement channel — restrict and monitor it

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Script won't run ("running scripts is disabled") | Execution policy — set an appropriate scope with `Set-ExecutionPolicy` (dev), or sign the script (prod) |
| Remoting fails to connect | WinRM not configured or firewall blocking 5985/5986 — run `Enable-PSRemoting` / check the listener |

## References

- [PowerShell documentation (Microsoft Learn)](https://learn.microsoft.com/en-us/powershell/)
- [About PowerShell logging / script block logging](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows)
- [Just Enough Administration (JEA)](https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows Commands](../Windows-Commands/Readme.md) — related module
- [Windows Monitoring and Logging](../Windows-Monitoring-and-Logging/Readme.md) — related module
- [Enterprise Security](../Enterprise-Security/Readme.md) — related module
