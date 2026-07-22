# FTP Logging

IIS FTP keeps its own logs, separate from the HTTP W3SVC logs, recording every connection, authentication, and file operation. Those logs are the primary source for detecting brute-force attempts, data exfiltration, and isolation-escape attempts against an FTP site.

## Overview

IIS FTP logs are written **separately from the HTTP W3SVC logs**, under a per-site folder named by the site's numeric ID, not its site name:

```text
%SystemDrive%\inetpub\logs\LogFiles\FTPSVC<SiteID>\
```

- Find the site ID in IIS Manager (site properties / General panel) or with:

```powershell
Get-Website "MyFTPSite" | Select-Object Name, Id   # untested
```

## Concepts

### Log Format and Fields

Logging uses the W3C Extended Log format. The relevant FTP-specific field set (as documented in the `<ftpServer><logFile>` config element) is:

`Date, Time, ClientIP, UserName, ServerIP, Method, UriStem, FtpStatus, Win32Status, ServerPort, FtpSubStatus, Session, FullPath, Info`

- Configure which fields are logged in **IIS Manager → select the FTP site → Logging** feature, or directly in `applicationHost.config` under `<site>/<ftpServer>/<logFile logExtFileFlags="...">`.

### Key FTP Status Codes

| Status | Meaning | Why it matters |
|---|---|---|
| `230` | User logged in | Successful authentication |
| `530` | Login incorrect / not logged in | Repeated occurrences = brute force |
| `RETR` (Method) | File download | Bulk `RETR` = possible exfiltration |
| `STOR` (Method) | File upload | Unexpected uploads = webshell/staging |

## Configuration

### GUI Steps

1. In IIS Manager, select the FTP site.
2. Double-click the **Logging** feature.
3. Set the log file directory, format (W3C), and rollover schedule.
4. Click **Select Fields** to choose the `logExtFileFlags` field set listed above.
5. Click **Apply**.

> [!NOTE]
> **Screenshot**
> ![IIS Manager FTP Logging feature page showing W3C format, log directory, and the Select W3C Fields dialog](placeholder-screenshot)

## Administration

### What to Watch

- Repeated `FtpStatus 530` (login incorrect) from the same `ClientIP` in a short window → credential brute-forcing. Cross-reference the source IP with firewall/perimeter logs.
- If FTP **Basic Authentication** validates against local or domain Windows accounts, failed/successful logons can also surface in the **Security** event log as network logons — see [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md) (Event IDs `4624`/`4625`) for the query pattern; confirm this correlation in your environment first, IIS FTP does not guarantee a 1:1 mapping for every authentication provider.
- Large numbers of `RETR` (`get`) operations in a short period from one account → possible bulk exfiltration.
- Any `UriStem` path that escapes a user's isolated home directory → possible isolation misconfiguration or path-traversal attempt (see [FTP-User-Isolation](FTP-User-Isolation.md)).
- Connections from IP ranges outside your expected client base, especially on an FTP site that should be internal-only.

## Examples

Quick scan of a log for failed logins by source IP (PowerShell):

```powershell
# untested
Select-String -Path "$env:SystemDrive\inetpub\logs\LogFiles\FTPSVC*\*.log" -Pattern " 530 " |
  ForEach-Object { ($_.Line -split ' ')[2] } |
  Group-Object | Sort-Object Count -Descending | Select-Object Count, Name
```

## Security Considerations

- Forward `FTPSVC*` logs to a central SIEM/log store so an attacker who compromises the host cannot trivially erase local evidence.
- Correlate FTP `530` spikes with Security-log `4625` events (see [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md)) to distinguish server-side FTP failures from broader account attacks.
- Protect the log directory with NTFS ACLs so FTP users cannot read or tamper with it.
- Pair log review with the preventive controls in [FTP-Security](FTP-Security.md) — detection without hardening only tells you after the fact.

## Troubleshooting

- **No `FTPSVC<SiteID>` folder appears** → logging disabled on the site, or the site has had no connections yet; enable Logging and reconnect.
- **Wrong site's logs** → folders are named by numeric site ID, not site name; resolve the ID with `Get-Website` as shown above.
- **Expected fields missing** → the `logExtFileFlags` field set excludes them; re-select fields in the Logging feature.

## References

- Microsoft Learn — [FTP Logging `<logFile>` element](https://learn.microsoft.com/en-us/iis/configuration/system.applicationhost/sites/site/ftpserver/logfile)
- Microsoft Learn — [FTP Status Codes in IIS 7 and later](https://learn.microsoft.com/en-us/troubleshoot/developer/webapps/iis/health-diagnostic-performance/ftp-status-codes-iis)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [FTP-Setup-in-IIS](FTP-Setup-in-IIS.md) — the FTP site whose activity is logged — related note
- [FTP-Security](FTP-Security.md) — the hardening/monitoring companion to this note — related note
- [FTP-User-Isolation](FTP-User-Isolation.md) — isolation-escape attempts that show up in these logs — related note
- [FTPS](FTPS.md) — TLS-handshake failures also surface in FTP logs — related note
- [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md) — Security-log `4624`/`4625` correlation for FTP logons — related note
