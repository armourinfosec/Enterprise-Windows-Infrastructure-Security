# FTP Server Administration

> [!NOTE]
> **Module hub**
> Installing, securing, and operating the built-in IIS FTP Server on Windows — from a basic file-transfer site to an encrypted, isolated, and audited enterprise deployment.

## Overview

This module covers the FTP publishing service that ships with the IIS role on Windows Server and Windows 10/11 Pro. It starts from a working plaintext FTP site and layers on the controls that make FTP acceptable in a managed environment: TLS encryption (FTPS), per-user isolation, hardening, and logging. SFTP (SSH-based transfer) is deliberately out of scope — the IIS FTP role does not provide it.

## Learning Objectives

- Install the IIS Web Server and FTP Server roles and create a functioning FTP site.
- Configure authentication, authorization, firewall/passive-port ranges, and users.
- Add SSL/TLS (explicit vs implicit FTPS) and understand how it differs from SFTP.
- Sandbox users to their own home directories with FTP User Isolation.
- Harden an FTP deployment against brute force, sniffing, and exfiltration.
- Locate, format, and interpret IIS FTP logs for detection and troubleshooting.

## Topics Covered

**Setup**

- [FTP-Setup-in-IIS](FTP-Setup-in-IIS.md) — install the IIS FTP role, create the site, configure auth/authorization/firewall, and add users.

**FTPS & Security**

- [FTPS](FTPS.md) — add SSL/TLS encryption; explicit vs implicit FTPS; certificate and SSL-policy configuration.
- [FTP-Security](FTP-Security.md) — consolidated hardening checklist and attack-surface reference.

**Isolation & Logging**

- [FTP-User-Isolation](FTP-User-Isolation.md) — confine each user to their own home directory; required folder layouts and isolation modes.
- [FTP-Logging](FTP-Logging.md) — where IIS FTP logs live, the W3C field set, and what to watch for.

## Practical Labs

- Stand up an IIS FTP site on a Windows Server VM, connect with `ftp.exe` and FileZilla, and confirm plaintext capture with a sniffer (Network-Sniffing).
- Enable Require-SSL FTPS with a self-signed certificate and re-test — verify the handshake and that plaintext is gone.
- Turn on physical-directory user isolation for two accounts and prove each account cannot escape its home folder.
- Generate failed and successful logins, then locate and parse the `FTPSVC<SiteID>` logs for the `530`/`230` events.

## Best Practices

- Prefer FTPS (or migrate to SFTP) over plain FTP on any untrusted network.
- Use passive mode with a narrow, firewalled passive-port range.
- Disable anonymous authentication unless the site is deliberately public.
- Enforce user isolation and least-privilege NTFS ACLs.
- Apply strong password policy; avoid shared accounts.
- Centralize and regularly review FTP logs.

## Security Considerations

FTP transmits credentials and data in cleartext by default and is a common brute-force and exfiltration target. Treat every plain-FTP site as exposed on untrusted networks: encrypt the transport, contain each account, restrict the network path, and monitor. See [FTP-Security](FTP-Security.md) for the full checklist and [FTP-Logging](FTP-Logging.md) for detection.

## Troubleshooting

- **530 Login incorrect** → wrong credentials or missing NTFS/authorization permissions.
- **Connection timeout / data channel not opening** → passive-port range not opened on the firewall and in FTP Firewall Support.
- **Cannot connect externally** → external IP / NAT not set in FTP Firewall Support.
- **FTPS handshake fails** → client explicit/implicit mode does not match the server binding (port 21 vs 990).
- **User lands in the wrong folder** → isolation folder missing or misnamed under `LocalUser\<name>`.

## References

- Microsoft Learn — [Installing and Configuring FTP on IIS](https://learn.microsoft.com/en-us/iis/get-started/whats-new-in-iis-8/installing-and-configuring-ftp-on-iis-8)
- Microsoft Learn — [FTP `<ftpServer>` configuration reference](https://learn.microsoft.com/en-us/iis/configuration/system.applicationhost/sites/site/ftpserver/)
- RFC 959 (FTP), RFC 4217 (FTP over TLS)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Internet Information Services (IIS)](../Web-Server-IIS/Internet-Information-Services(IIS).md) — the role that hosts the FTP service — related module
- [Types-of-Site-Binding-in-IIS](../Web-Server-IIS/Types-of-Site-Binding-in-IIS.md) — bindings used to expose FTP (including implicit-FTPS port 990) — related note
- [Authentication-Methods-in-Windows](../Web-Server-IIS/Authentication-Methods-in-Windows.md) — auth backing FTP Basic Authentication — related note
- [Windows-Firewall-and-AV-Commands](../Windows-Commands/Windows-Firewall-and-AV-Commands.md) — firewall syntax for FTP rules — related note
- File-Transfers — cross-course index of file-transfer techniques — related note
