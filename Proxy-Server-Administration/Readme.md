# Proxy Server Administration

Sitting a proxy between clients and the internet to control, cache, and hide traffic — and the NAT and port-forwarding that shape how hosts reach each other.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

A proxy server brokers traffic on behalf of clients: it can filter and log requests, cache responses, and mask the originating host. This module covers proxy concepts and types (forward, reverse, transparent, anonymous), a concrete Windows proxy (CCProxy), and the closely related network-edge mechanisms — Network Address Translation and port forwarding — that determine which internal services are reachable from outside.

## Learning Objectives

By the end of this module you will be able to:

- Explain what a proxy server does and distinguish forward, reverse, transparent, and anonymous proxies
- Deploy a basic Windows proxy (CCProxy) for client traffic control
- Describe how NAT and port forwarding expose or hide internal services at the network edge

## Topics Covered

This module contains **5 notes**.

| Note | Topic |
| --- | --- |
| [Proxy-Servers](Proxy-Servers.md) | Proxy server concepts and purpose |
| [Types-of-Proxies](Types-of-Proxies.md) | Forward, reverse, transparent, anonymous |
| [CCProxy](CCProxy.md) | Deploying CCProxy on Windows |
| [Network-Address-Translation(NAT)](Network-Address-Translation(NAT).md) | NAT and address translation |
| [Port-Forwarding](Port-Forwarding.md) | Exposing internal services via port forwarding |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, stand up CCProxy (or Squid) on a lab server, point a client's browser at it, and observe request logging; then configure a port-forward to publish an internal web server and test reachability from the attacker VM.

## Best Practices

- Authenticate and log proxy clients; a proxy that anyone can use anonymously is an open relay
- Forward only the specific ports a service needs, and restrict source addresses where possible
- Use a reverse proxy to terminate TLS and shield origin servers rather than exposing them directly

## Security Considerations

> [!WARNING]
> **Proxies concentrate trust — and traffic**
> Every session flows through the proxy, which makes it both a powerful control point and a high-value target.

- Open/misconfigured proxies enable abuse, pivoting, and traffic laundering — lock down ACLs
- Port forwarding punches holes in the perimeter; each forward is new attack surface to inventory
- NAT is not a security control — internal hosts still need host-level hardening

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Clients can't browse through the proxy | Wrong proxy address/port in client config, or proxy ACL is blocking the source subnet |
| Forwarded service unreachable from outside | NAT/port-forward rule missing or firewall blocking — verify the mapping and allow the inbound port |

## References

- [What is a proxy server? (Cloudflare Learning)](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/)
- [RFC 3022 — Traditional IP Network Address Translator (NAT)](https://www.rfc-editor.org/rfc/rfc3022)
- [OWASP — Reverse proxy considerations](https://owasp.org/www-community/vulnerabilities/)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Networking Fundamentals](../Networking-Fundamentals/Readme.md) — related module
- [Remote Access and VPN Configuration](../Remote-Access-and-VPN-Configuration/Readme.md) — related module
