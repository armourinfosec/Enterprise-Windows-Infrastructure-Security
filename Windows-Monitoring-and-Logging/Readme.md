# Windows Monitoring and Logging

Turning a Windows estate from opaque to observable — the audit policy, event logs, Sysmon, and forwarding that let you detect and reconstruct attacks.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

You cannot defend what you cannot see. This module covers the Windows telemetry stack: the audit policy that decides what gets recorded, the Event Log channels and the key event IDs that matter for security, Sysmon for high-fidelity endpoint telemetry, and Windows Event Forwarding/Collection (WEF/WEC) to centralize logs off the hosts that generate them. It ties the per-module detection notes across the course into one coherent monitoring discipline and points toward SIEM integration.

## Learning Objectives

By the end of this module you will be able to:

- Configure advanced audit policy to capture logon, privilege-use, process-creation, and object-access events
- Read and interpret the security-relevant Windows event IDs and query them with `Get-WinEvent`
- Deploy Sysmon and centralize logs with Windows Event Forwarding for detection and incident response

## Topics Covered

This module contains **7 notes**.

| Note | Topic |
| --- | --- |
| [Windows-Advanced-Audit-Policy](Windows-Advanced-Audit-Policy.md) | Configuring advanced audit subcategories |
| [Key-Security-Event-IDs](Key-Security-Event-IDs.md) | The event IDs that matter (4624/4625/4688/4720/1102/…) |
| [Querying-Logs-with-Get-WinEvent](Querying-Logs-with-Get-WinEvent.md) | Filtering and hunting in event logs |
| [Sysmon-Deployment-and-Configuration](Sysmon-Deployment-and-Configuration.md) | High-fidelity endpoint telemetry with Sysmon |
| [Windows-Event-Forwarding-WEF-WEC](Windows-Event-Forwarding-WEF-WEC.md) | Centralizing logs off-host |
| [Command-Line-and-Process-Auditing](Command-Line-and-Process-Auditing.md) | Script-block, process-creation, and command-line auditing |
| [SIEM-Integration](SIEM-Integration.md) | Shipping and correlating Windows telemetry |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection. Planned exercises include enabling process-creation auditing with command line, deploying Sysmon with a curated config, and forwarding a client's Security log to a collector.

## Best Practices

- Enable advanced audit policy via GPO with a documented baseline — default auditing misses most attacks
- Turn on process-creation (4688) auditing **with** command-line capture, and deploy Sysmon for depth
- Forward logs to a central collector/SIEM so a wiped endpoint doesn't erase the evidence

## Security Considerations

> [!WARNING]
> **Logs are an attacker target**
> Clearing and tampering with logs is a standard post-exploitation step — treat log integrity as a control, not an afterthought.

- Alert on `1102` (Security log cleared) and Sysmon/WEF service tampering
- Keep collected logs on a system the endpoint's credentials cannot reach or modify
- Balance verbosity vs noise — over-collecting without tuning buries the signals that matter

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Expected events aren't appearing | Audit subcategory not enabled — set via `auditpol /set` or Advanced Audit Policy GPO (and beware SACLs for object access) |
| Forwarded events not reaching the collector | WinRM/WEF subscription or channel-access misconfig — check the subscription, source-initiated config, and `Network Service` log read rights |

## References

- [Advanced security audit policy (Microsoft Learn)](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/advanced-security-audit-policy-settings)
- [Sysmon (Sysinternals)](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Windows Event Forwarding (Microsoft Learn)](https://learn.microsoft.com/en-us/windows/security/threat-protection/use-windows-event-forwarding-to-assist-in-intrusion-detection)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows Operating System Administration](../Windows-Operating-System-Administration/Readme.md) — related module
- [Windows PowerShell](../Windows-PowerShell/Readme.md) — related module
- [Enterprise Security](../Enterprise-Security/Readme.md) — related module
