# Software Development Life Cycle

How software and websites are built and hosted — the development context that infrastructure exists to serve and secure.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Infrastructure does not exist for its own sake — it hosts applications and websites produced by a development process. This short module frames that context: the phases of the Software Development Life Cycle and the IT roles around it, the basics of how websites are built (HTML/CSS), what a website is, and how it is hosted. Understanding this pipeline clarifies where security fits and why the web-server and IIS modules matter.

## Learning Objectives

By the end of this module you will be able to:

- Describe the phases of the SDLC and the IT roles involved at each stage
- Explain what a website is and the role of HTML/CSS in building one
- Understand hosting models and how they connect to the web-server modules

## Topics Covered

This module contains **4 notes**.

| Note | Topic |
| --- | --- |
| [Software-Development-Life-Cycle(SDLC)-and-Related-IT-Roles](Software-Development-Life-Cycle(SDLC)-and-Related-IT-Roles.md) | SDLC phases and IT roles |
| [Website](Website.md) | What a website is |
| [HTML-and-CSS](HTML-and-CSS.md) | Front-end building blocks |
| [Hosting](Hosting.md) | Website hosting models |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, build a minimal static site with HTML/CSS and host it on the IIS server from the Web-Server module — connecting the development and hosting halves end to end.

## Best Practices

- Build security into every SDLC phase (secure design, code review, testing) rather than bolting it on at the end
- Separate development, staging, and production hosting environments
- Keep front-end and hosting configuration in version control for repeatability

## Security Considerations

> [!WARNING]
> **Security is a life-cycle property, not a final step**
> Vulnerabilities introduced early (design, dependencies) are cheapest to fix early and most expensive in production.

- Insecure defaults in hosting (directory listing, verbose errors) leak information — harden before publishing
- Track third-party dependencies for known vulnerabilities across the life cycle
- Treat the hosting server as part of the application's trust boundary

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Site renders locally but not when hosted | Missing files, wrong document root, or MIME/handler config on the web server |
| Styling doesn't load | Broken relative paths or blocked static-content handler — check the CSS reference and server config |

## References

- [MDN Web Docs — Getting started with the web](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web)
- [OWASP Secure Software Development Life Cycle](https://owasp.org/www-project-integration-standards/writeups/owasp_in_sdlc/)
- [NIST SSDF (SP 800-218)](https://csrc.nist.gov/pubs/sp/800/218/final)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Web Server (IIS)](../Web-Server-IIS/Readme.md) — related module
- [Software-Development-Life-Cycle(SDLC)-and-Related-IT-Roles](Software-Development-Life-Cycle(SDLC)-and-Related-IT-Roles.md) — start here
