# Domain Name System (DNS)

> [!NOTE]
> **Module hub**
> The Domain Name System (DNS) module covers how human-readable names resolve to IP addresses, how zones, records, and server roles are structured, and how caching, forwarding, dynamic updates, and DNSSEC are administered in a Windows Server DNS deployment.

## Overview

DNS is the distributed, hierarchical naming service that underpins nearly every network interaction — web, email, Active Directory, and service discovery all depend on it. This module builds a complete picture of DNS from the global root/TLD/authoritative hierarchy down to the individual resource records in a zone, then moves into operating DNS as a Windows Server role: primary and secondary servers, recursive resolvers, forwarders and conditional forwarders, client- and server-side caching, dynamic DNS, split-horizon views, and DNSSEC signing. Offensive and defensive angles (zone transfers, cache poisoning, information disclosure) are called out throughout, since DNS is both a reconnaissance goldmine and a common attack surface.

## Learning Objectives

- Explain the DNS resolution flow through root, TLD, and authoritative name servers.
- Identify and use the common resource-record types (A, AAAA, CNAME, MX, TXT, NS, SOA, PTR, SRV, CAA).
- Distinguish forward and reverse lookup zones and how `in-addr.arpa` / `ip6.arpa` work.
- Deploy and differentiate primary, secondary, recursive/caching, and forwarding DNS server roles.
- Configure forwarders and conditional forwarders for upstream and domain-specific resolution.
- Manage client and server DNS caches and troubleshoot stale or poisoned entries.
- Automate zone and record creation with PowerShell.
- Secure DNS with DNSSEC and split-horizon (split-brain) DNS.
- Perform WHOIS lookups and reason about domain structure, TLDs, and valuation.

## Topics Covered

### Concepts & Hierarchy

- [DNS-Hierarchy-and-How-It-Works](DNS-Hierarchy-and-How-It-Works.md) — root, TLD, and authoritative resolution flow
- [Domain-Name-Structure](Domain-Name-Structure.md) — anatomy of a domain namespace
- [Fully-Qualified-Domain-Name(FQDN)](Fully-Qualified-Domain-Name(FQDN).md) — absolute names and the trailing dot

### Zones & Records

- [DNS-Records-and-Their-Types](DNS-Records-and-Their-Types.md) — A, AAAA, CNAME, MX, TXT, NS, SOA, PTR, SRV, CAA
- [Forward-and-Reverse-DNS-Zones](Forward-and-Reverse-DNS-Zones.md) — name-to-IP and IP-to-name zones
- [PowerShell-script-to-create-DNS-zones](PowerShell-script-to-create-DNS-zones.md) — scripting zone and record creation

### Server Types & Forwarders

- [DNS Server Types and Their Roles](DNS-Server-Types.md) — overview of server roles
- [Primary-(Master)-DNS-Server](Primary-(Master)-DNS-Server.md) — authoritative primary server
- [Secondary-(Slave)-DNS-Server](Secondary-(Slave)-DNS-Server.md) — replicated secondary server and zone transfers
- [Recursive-(Caching)-DNS-Server](Recursive-(Caching)-DNS-Server.md) — recursive/caching resolver
- [Forwarders-Nameserver](Forwarders-Nameserver.md) — forwarding queries upstream
- [Conditional-Forwarders-in-DNS](Conditional-Forwarders-in-DNS.md) — domain-specific forwarding

### Caching

- [DNS-Cache](DNS-Cache.md) — client-side resolver cache
- [DNS-Server-Cache](DNS-Server-Cache.md) — server-side query cache

### Dynamic DNS & Security

- [Dynamic-DNS-(DDNS)](Dynamic-DNS-(DDNS).md) — automatic record updates
- [DNSSEC](DNSSEC.md) — signing zones and building the chain of trust
- [Split-DNS](Split-DNS.md) — split-horizon internal vs external views

### Domains & WHOIS

- [Whois](Whois.md) — WHOIS domain and registration lookups
- [Domain-Appraisals](Domain-Appraisals.md) — estimating the value of a domain name
- [.shop Top-Level Domain](dot-shop-Top-Level-Domain.md) — a specific TLD example
- [Hosts-File](Hosts-File.md) — local name-resolution override

## Practical Labs

- Stand up the Windows DNS Server role and create a forward and reverse lookup zone (see [Forward-and-Reverse-DNS-Zones](Forward-and-Reverse-DNS-Zones.md) and [PowerShell-script-to-create-DNS-zones](PowerShell-script-to-create-DNS-zones.md)).
- Configure a secondary server and observe an AXFR/IXFR zone transfer (see [Secondary-(Slave)-DNS-Server](Secondary-(Slave)-DNS-Server.md)).
- Add forwarders and a conditional forwarder, then trace resolution with `nslookup` / `dig` (see [Forwarders-Nameserver](Forwarders-Nameserver.md), [Conditional-Forwarders-in-DNS](Conditional-Forwarders-in-DNS.md)).
- Sign a zone with DNSSEC and validate the chain of trust (see [DNSSEC](DNSSEC.md)).
- Build a split-horizon deployment with internal and external views of one zone (see [Split-DNS](Split-DNS.md)).

## Best Practices

- Keep at least one secondary server for every authoritative zone for redundancy.
- Restrict zone transfers to named secondary servers only.
- Use forwarders and conditional forwarders instead of full recursion where policy or filtering is required.
- Enable DNSSEC on internet-facing authoritative zones and validation on resolvers.
- Use split-horizon DNS so internal hostnames are never exposed to external queries.
- Set TTLs deliberately — short for records that change often, long to reduce query load.

## Security Considerations

- **Zone transfers** — unrestricted AXFR leaks the entire zone to attackers; lock down transfers.
- **Cache poisoning / spoofing** — mitigate with DNSSEC validation, pollution protection, and source-port randomization.
- **Information disclosure** — internal names, subdomains, and infrastructure leak via DNS cache, WHOIS, and misconfigured split-DNS.
- **DNS as a recon and exfil channel** — cached entries and records reveal internal services; DNS tunneling can exfiltrate data.

## Troubleshooting

- Use `ipconfig /displaydns`, `ipconfig /flushdns`, and `Get-DnsServerCache` to inspect and clear caches.
- Trace resolution end to end with `nslookup`, `dig +trace`, and per-tier queries.
- Verify zone transfers and SOA serial increments when secondaries fall out of sync.
- Confirm DNSSEC signatures (RRSIG/DNSKEY/DS) when validation fails.

## References

- [Verisign: How DNS Works](https://www.verisign.com/en_US/website-presence/online/how-dns-works/index.xhtml)
- [ICANN / IANA Root Server Information](https://www.iana.org/domains/root/servers)
- [Cloudflare DNS Learning Center](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [Microsoft Learn: DNS on Windows Server](https://learn.microsoft.com/en-us/windows-server/networking/dns/dns-top)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Dynamic Host Configuration Protocol (DHCP)](../Dynamic-Host-Configuration-Protocol-DHCP/Readme.md) — sibling addressing service module
- [Active Directory Domain Services (AD DS)](../Active-Directory-Domain-Services-AD-DS/Readme.md) — depends on DNS for locator records
