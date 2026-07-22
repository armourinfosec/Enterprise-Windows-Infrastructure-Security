# File Transfers

> Overview of the common protocols and methods used to move files between systems in an enterprise, and how they compare on security.

## Overview

Enterprises move files constantly — between servers, to clients, and across sites. Choosing the right transfer method is a trade-off between **compatibility**, **performance**, and **security**. This note indexes the main options and points to the detailed notes for each.

## Protocol Comparison

| Method | Transport | Default Port | Encryption | Notes |
|--------|-----------|--------------|------------|-------|
| **FTP** | TCP | 21 (control) | ❌ None | Credentials and data in cleartext — avoid on untrusted networks |
| **FTPS** | TCP + TLS | 21 / 990 | ✅ TLS | FTP wrapped in TLS |
| **SFTP** | SSH | 22 | ✅ SSH | File transfer subsystem of SSH; single connection |
| **SCP** | SSH | 22 | ✅ SSH | Simple copy over SSH |
| **TFTP** | UDP | 69 | ❌ None | Minimal, no auth — used for PXE/network boot |
| **SMB** | TCP | 445 | ✅ (SMB3) | Windows file sharing; supports encryption in SMB3 |
| **HTTP/HTTPS** | TCP | 80 / 443 | HTTPS only | Web download/upload; use HTTPS |

## Choosing a Method

- **Prefer encrypted protocols** (SFTP, FTPS, SMB3, HTTPS) for anything crossing untrusted networks.
- Use **SFTP or SCP** where SSH is already available — one port, strong auth options.
- Reserve **TFTP** for isolated provisioning networks (e.g. PXE boot); never expose it broadly.
- For internal Windows file sharing, use **SMB with signing and SMB3 encryption**.

> [!TIP]
> If you must interoperate with legacy FTP, put it behind FTPS and restrict it to trusted network segments — see why in [Network-Sniffing](Network-Sniffing.md).

## Security Considerations

- Cleartext protocols (FTP, TFTP, plain HTTP) expose credentials and data to anyone sniffing the network.
- Enforce strong authentication and, where possible, key-based auth (SSH).
- Log transfers and monitor for unusual volumes or destinations.

## References

- OpenSSH SFTP — <https://man.openbsd.org/sftp>
- Microsoft SMB security — <https://learn.microsoft.com/windows-server/storage/file-server/smb-security>

## Related

- [Network-Sniffing](Network-Sniffing.md)
- **Secure-FTP-FTPS-and-SFTP**
