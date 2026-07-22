# Web Server (IIS)

Hosting web applications on Windows with Internet Information Services — bindings, PHP/MySQL stacks, and the authentication model that guards them.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Internet Information Services (IIS) is the Windows web server role. This module covers standing up IIS, the site-binding types that map requests to sites (IP, port, host header, HTTPS), running a PHP/MySQL application on Windows (including phpMyAdmin), and the Windows authentication model behind it all — the authentication methods IIS supports and the crucial distinction between authentication and authorization.

## Learning Objectives

By the end of this module you will be able to:

- Install IIS and configure site bindings by IP, port, host header, and HTTPS
- Run a PHP/MySQL application (with phpMyAdmin) on a Windows/IIS host
- Explain the IIS authentication methods and the difference between authentication and authorization

## Topics Covered

This module contains **6 notes**.

| Note | Topic |
| --- | --- |
| [Internet-Information-Services(IIS)](Internet-Information-Services(IIS).md) | IIS overview and setup |
| [Types-of-Site-Binding-in-IIS](Types-of-Site-Binding-in-IIS.md) | IP, port, host-header, and HTTPS bindings |
| [Setting-Up-PHP-on-Windows-Server](Setting-Up-PHP-on-Windows-Server.md) | Running PHP on Windows/IIS |
| [phpMyAdmin-on-Windows-Server-with-IIS](phpMyAdmin-on-Windows-Server-with-IIS.md) | Deploying phpMyAdmin with IIS |
| [Authentication-Methods-in-Windows](Authentication-Methods-in-Windows.md) | IIS/Windows authentication methods |
| [Authentication-vs-Authorization](Authentication-vs-Authorization.md) | Authentication vs authorization |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, install the IIS role, publish two sites separated by host header, add an HTTPS binding with a self-signed cert, then layer a PHP/phpMyAdmin app and test each authentication method.

## Best Practices

- Remove unused IIS modules and default content; run app pools under least-privilege identities
- Bind HTTPS with modern TLS and redirect HTTP; use distinct app pools per site for isolation
- Keep PHP and phpMyAdmin patched and restrict phpMyAdmin to trusted networks

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> A default IIS + PHP stack ships with information-leaking defaults that are prime targets on the open internet.

- Disable directory browsing and verbose errors; hide server/version banners
- phpMyAdmin is a frequent breach vector — never expose it publicly without auth and IP restriction
- Enforce authorization checks server-side; authentication alone does not restrict what a user may access

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Requests hit the wrong site | Overlapping bindings — make host header/port/IP unique per site and check binding precedence |
| PHP pages download instead of executing | PHP handler mapping missing in IIS — register the FastCGI/PHP handler for the site |

## References

- [IIS documentation (Microsoft Learn)](https://learn.microsoft.com/en-us/iis/)
- [OWASP — Securing web servers](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)
- [PHP on IIS](https://learn.microsoft.com/en-us/iis/application-frameworks/install-and-configure-php-on-iis/)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Software Development Life Cycle](../Software-Development-Life-Cycle/Readme.md) — related module
- [Windows Server Management](../Windows-Server-Management/Readme.md) — related module
