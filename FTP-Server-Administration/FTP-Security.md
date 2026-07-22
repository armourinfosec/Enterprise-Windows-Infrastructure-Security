# FTP Security

FTP is cleartext by default, which makes an IIS FTP site one of the higher-risk services to expose. This note is the consolidated hardening and detection reference for FTP on Windows/IIS — how to shrink its attack surface and how to catch abuse.

## Overview

FTP is **cleartext by default**: credentials and file contents can be captured by anyone positioned on the network path (see Network-Sniffing). Treat any plain-FTP deployment as inherently exposed on untrusted networks. Hardening is layered: encrypt the transport, disable what you don't use, contain each account, restrict the network path, and monitor.

> [!WARNING]
> **Assume plaintext FTP is compromised on untrusted networks**
> If a site must be reachable from outside a trusted LAN, it should be FTPS (or migrated to SFTP), firewall-restricted, and monitored — not plain FTP.

## Concepts

### Attack Surface

| Weakness | Impact | Mitigation |
|---|---|---|
| Cleartext control + data channels | Credential and data capture, MITM | [FTPS](FTPS.md) (Require SSL) |
| Anonymous authentication | Unauthenticated read/write | Disable unless deliberately public |
| Shared / default accounts | Credential reuse, weak attribution | Per-purpose accounts, strong passwords |
| No user isolation | One account browses the whole tree | [FTP-User-Isolation](FTP-User-Isolation.md) |
| Wide-open passive port range / source IPs | Brute force, exposure | Firewall narrowing, Dynamic IP Restrictions |
| No logging review | Undetected brute force / exfil | [FTP-Logging](FTP-Logging.md) |

## Security Considerations

### Hardening Checklist

- **Disable Anonymous Authentication** unless the site is explicitly meant to be public/anonymous — confirm this in the FTP Authentication feature.
- **Prefer FTPS** (see [FTPS](FTPS.md)) for anything crossing an untrusted network; consider migrating to an SSH-based SFTP/SCP service instead of IIS FTP where the client base allows it.
- **Enforce FTP User Isolation** (see [FTP-User-Isolation](FTP-User-Isolation.md)) so a compromised FTP account cannot browse or write outside its own directory tree.
- **Restrict at the firewall** — only allow the control port and a narrow passive-port range from known source IP ranges where feasible:

```cmd
netsh advfirewall firewall add rule name="FTP Control (990 Implicit FTPS)" dir=in action=allow enable=yes profile=domain protocol=TCP localport=990
:: untested
```

- **Enable Dynamic IP Restrictions** (IIS feature, installable alongside the FTP role) to auto-block a source IP after a configurable number of failed logon attempts in a time window — a lightweight brute-force mitigation for internet-facing FTP sites.   # untested — verify feature availability/exact UI path on your Windows Server build before relying on this
- **Apply strong password policy** and disable/rename any default or shared FTP accounts; use dedicated per-purpose accounts, not shared credentials.

## Best Practices

- Prefer **SFTP or FTPS** over plain FTP.
- Use **Passive Mode** with a narrow, firewalled port range for NAT/firewall compatibility without over-exposing ports.
- Restrict users to **their own directories** via isolation.
- Apply **strong password policies** and disable anonymous access.
- Use **Windows Groups** for easier, auditable ACL management.
- Periodically re-review authorization rules — remove any temporary "Allow All Users" rules left over from testing.

## Detection and Monitoring

- Watch `FTPSVC*` logs (see [FTP-Logging](FTP-Logging.md)) for repeated `530` failures, unexpected source IPs, and abnormal transfer volume.
- Where FTP authentication maps to Windows accounts, correlate with Security-log `4624`/`4625` via [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md).
- Periodically audit **FTP Authorization Rules** to confirm no stale "Allow All Users" rules have crept back in after testing.

> [!TIP]
> **Firewall command reference**
> The `netsh advfirewall` and inbound-rule syntax used above is covered in depth in [Windows-Firewall-and-AV-Commands](../Windows-Commands/Windows-Firewall-and-AV-Commands.md).

## Troubleshooting

- **Legitimate users blocked after enabling Dynamic IP Restrictions** → threshold too aggressive or shared NAT egress IP; raise the failed-attempt threshold or allowlist known ranges.
- **FTPS enforced but clients still connect in plaintext** → SSL policy set to "Allow" rather than "Require"; switch to Require (see [FTPS](FTPS.md)).
- **Anonymous access still working after "disabling" it** → an Authorization rule still grants "All Users"; remove or scope the rule.

## References

- Microsoft Learn — [FTP Security (Authentication, Authorization, SSL)](https://learn.microsoft.com/en-us/iis/configuration/system.applicationhost/sites/site/ftpserver/security/)
- Microsoft Learn — [Dynamic IP Restrictions for FTP](https://learn.microsoft.com/en-us/iis/configuration/system.ftpserver/security/dynamicipsecurity)
- OWASP — Transport Layer Protection Cheat Sheet

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [FTP-Setup-in-IIS](FTP-Setup-in-IIS.md) — the base FTP site this hardening is applied to — related note
- [FTPS](FTPS.md) — the transport-encryption control at the top of the checklist — related note
- [FTP-User-Isolation](FTP-User-Isolation.md) — the containment control in the checklist — related note
- [FTP-Logging](FTP-Logging.md) — the detection/monitoring companion to this note — related note
- [Windows-Firewall-and-AV-Commands](../Windows-Commands/Windows-Firewall-and-AV-Commands.md) — firewall command syntax used here — related note
- Network-Sniffing — the cleartext-capture threat driving these controls — related note
- [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md) — Security-log correlation for FTP logons — related note
