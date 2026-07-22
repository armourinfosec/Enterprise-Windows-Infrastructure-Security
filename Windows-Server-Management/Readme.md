# Windows Server Management

Standing up and administering Windows Server itself — editions, roles, services, and the remote-management channels (WinRM, OpenSSH) you drive it through.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Windows Server is the platform every infrastructure role in this course runs on. This module covers the server itself: editions and hardware considerations, the Computer Management console, the Windows service model, and — most importantly for administration at scale — remote management via Windows Remote Management (WinRM) and OpenSSH on Windows. These are the entry points an administrator uses daily and that an attacker prizes for lateral movement.

## Learning Objectives

By the end of this module you will be able to:

- Compare Windows Server editions and size server hardware for a role
- Manage services and the host via the Computer Management console and the service model
- Enable and use remote management over WinRM and OpenSSH securely

## Topics Covered

This module contains **10 notes**.

| Note | Topic |
| --- | --- |
| [Windows-Server](Windows-Server.md) | Windows Server overview |
| [Windows-Server-Editions](Windows-Server-Editions.md) | Server editions and licensing |
| [Server-Hardware](Server-Hardware.md) | Server hardware considerations |
| [Computer-Management-in-Windows-OS](Computer-Management-in-Windows-OS.md) | The Computer Management console |
| [Windows-Service](Windows-Service.md) | The Windows service model |
| [Windows-Remote-Management(WinRM)](Windows-Remote-Management(WinRM).md) | Remote management via WinRM |
| [OpenSSH-Server-on-Windows](OpenSSH-Server-on-Windows.md) | OpenSSH server on Windows |
| [Server-Manager](Server-Manager.md) | Server Manager console |
| [Windows-Features-and-Roles](Windows-Features-and-Roles.md) | Installing features and roles |
| [Windows-Server-Installation](Windows-Server-Installation.md) | Windows Server installation |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, deploy Windows Server in the lab, enable WinRM and OpenSSH, then manage the host remotely from both a Windows admin box (`Enter-PSSession`) and a Kali box (`ssh`), comparing the two channels.

## Best Practices

- Use Server Core where a GUI isn't needed to shrink attack surface and patch footprint
- Restrict WinRM/OpenSSH to management subnets and require strong auth (Kerberos, key-based SSH)
- Run services under least-privilege service accounts (gMSA where possible) and document enabled roles

## Security Considerations

> [!WARNING]
> **Remote management is remote code execution**
> WinRM and SSH are, by design, ways to run commands on the server — anyone who can reach and authenticate to them owns the host.

- WinRM (5985/5986) is a primary lateral-movement vector — restrict it and prefer HTTPS/5986 with valid certs
- Audit service configurations for unquoted paths and weak permissions (privilege-escalation classics)
- Key-based SSH beats passwords; protect and rotate the `administrators_authorized_keys` file

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| `Enter-PSSession` fails to connect | WinRM not enabled/listener missing or firewall blocking 5985/5986 — run `winrm quickconfig` and check the listener |
| OpenSSH login as admin fails with keys | Admin keys must live in `%ProgramData%\ssh\administrators_authorized_keys` with correct ACLs, not the user profile |

## References

- [Windows Server documentation (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/)
- [Windows Remote Management (WinRM)](https://learn.microsoft.com/en-us/windows/win32/winrm/portal)
- [OpenSSH in Windows](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows Operating System Administration](../Windows-Operating-System-Administration/Readme.md) — related module
- [Remote Access and VPN Configuration](../Remote-Access-and-VPN-Configuration/Readme.md) — related module
