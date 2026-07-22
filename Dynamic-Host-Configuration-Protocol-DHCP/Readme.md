# Dynamic Host Configuration Protocol (DHCP)

Automatic IP-address assignment for a network — how leases are handed out and configured, and the attacks and defenses that target the protocol.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

DHCP assigns IP addresses, subnet masks, gateways, and DNS servers to clients automatically, removing the need for manual configuration. This module walks the protocol from the DORA handshake through the full server configuration surface — scopes, exclusions, reservations, scope/server options, policies, filters, and cross-subnet relay — and then turns to security: the starvation, rogue-server, and man-in-the-middle attacks the protocol enables, and the switch-level `DHCP snooping` defense that stops them.

## Learning Objectives

By the end of this module you will be able to:

- Explain the DORA lease process and configure scopes, exclusions, reservations, and options on a Windows DHCP server
- Extend DHCP across subnets with a relay agent (IP helper) and apply policies and MAC filters
- Recognize DHCP starvation and rogue-server attacks and mitigate them with snooping and filtering

## Topics Covered

This module contains **15 notes**.

| Note | Topic |
| --- | --- |
| [DHCP(Dynamic-Host-Configuration-Protocol)](DHCP(Dynamic-Host-Configuration-Protocol).md) | DHCP protocol overview |
| [DORA-Process](DORA-Process.md) | Discover, Offer, Request, Acknowledge handshake |
| [Scope-in-a-DHCP-Server](Scope-in-a-DHCP-Server.md) | Creating and managing scopes |
| [DHCP-Scope-Options](DHCP-Scope-Options.md) | Per-scope options |
| [DHCP-Server-Options](DHCP-Server-Options.md) | Server-wide options |
| [Exclusion-Range-in-DHCP](Exclusion-Range-in-DHCP.md) | Addresses excluded from a scope |
| [DHCP-Reservations](DHCP-Reservations.md) | MAC-bound fixed IP addresses |
| [DHCP-Policies](DHCP-Policies.md) | Conditional address assignment |
| [DHCP-Filters-Allow-and-Deny](DHCP-Filters-Allow-and-Deny.md) | MAC allow/deny lists |
| [DHCP-Relay-Agent-IP-Helper](DHCP-Relay-Agent-IP-Helper.md) | Forwarding DHCP across subnets |
| [DHCP-Security-Issues-and-Attacks](DHCP-Security-Issues-and-Attacks.md) | DHCP attack surface overview |
| [DHCP-Starvation-Attack](DHCP-Starvation-Attack.md) | Pool-exhaustion denial of service |
| [Rogue-DHCP-Server](Rogue-DHCP-Server.md) | Malicious server / man-in-the-middle |
| [DHCP-Snooping](DHCP-Snooping.md) | Switch-level mitigation |
| [DHCP-High-Availability-Failover](DHCP-High-Availability-Failover.md) | DHCP failover (Load Balance / Hot Standby) |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, install the DHCP role on a lab server, author a scope with an exclusion and a reservation, then run a starvation attack (e.g. `dhcpstarv` / Yersinia) from a client VM and watch the pool drain — the fastest way to internalize why snooping matters.

## Best Practices

- Size scopes with headroom and set lease durations to match device churn (short for guest/Wi-Fi, long for static workstations)
- Use reservations for servers and printers instead of static addresses so DNS/option changes propagate centrally
- Authorize DHCP servers in AD and document scope options in one place

## Security Considerations

> [!WARNING]
> **DHCP is unauthenticated by design**
> The protocol has no built-in authentication — a client trusts whichever server answers first, which is exactly what attackers abuse.

- Enable **DHCP snooping** on access switches so only trusted ports may offer leases (kills rogue servers)
- Watch for starvation attacks that exhaust the pool and enable a rogue server to take over
- Combine with Dynamic ARP Inspection and IP Source Guard to close the man-in-the-middle path

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Clients get 169.254.x.x (APIPA) addresses | No DHCP reply reaching them — check the server scope is active/authorized and that relay/IP-helper is set on remote subnets |
| Address conflicts on the LAN | A rogue or second DHCP server is answering — locate it via `ipconfig /all` gateway/DNS mismatch and enable snooping |

## References

- [DHCP overview (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/networking/technologies/dhcp/dhcp-top)
- [RFC 2131 — Dynamic Host Configuration Protocol](https://www.rfc-editor.org/rfc/rfc2131)
- [Cisco — DHCP Snooping configuration guide](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/17-x/configuration_guide/sec/b_17x_sec_9300_cg/configuring_dhcp_features_and_option_82.html)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Domain Name System (DNS)](../Domain-Name-System-DNS/Readme.md) — sibling name-resolution service
- [Networking Fundamentals](../Networking-Fundamentals/Readme.md) — related module
