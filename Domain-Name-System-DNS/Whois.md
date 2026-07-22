# WHOIS

**WHOIS** is an Internet service and protocol used to query databases that store information about the registration of **domain names, IP addresses, and Autonomous System Numbers (ASNs)**. It is widely used by system administrators, security researchers, and legal professionals to identify the ownership and administrative details of internet resources.

## Overview

A WHOIS record works like a **public directory entry for a domain or IP resource**, providing transparency about who registered it, which organization manages it, and when it was created or will expire.

> [!NOTE]
> **WHOIS vs RDAP**
> WHOIS is the legacy protocol (TCP port 43, unstructured text). The registry industry is transitioning to **RDAP**, which returns structured JSON — see the RDAP section below.

## Concepts

### What Information Can WHOIS Provide?

A typical WHOIS lookup may reveal the following details.

#### Domain Information

- **Domain Name** — the queried domain (e.g., `example.com`).
- **Domain Status** — indicates the state of the domain in the registry, such as:
  - `active`
  - `clientTransferProhibited`
  - `pendingDelete`
  - `redemptionPeriod`
  - `expired`

#### Registrar Information

The **company responsible for registering and managing the domain**. Examples: GoDaddy, Namecheap, Cloudflare, Google Domains.

Typical fields include:

- Registrar Name
- Registrar IANA ID
- Registrar Website
- Abuse Contact Email
- Abuse Contact Phone

#### Registrant Information

Details about the **domain owner or organization** that registered the domain. This may include:

- Registrant Name
- Organization
- Address
- City / State / Country
- Email Address
- Phone Number

> [!NOTE]
> **Privacy protection**
> In many cases this information is **hidden using WHOIS Privacy Protection** or **Proxy Services**.

#### Important Dates

WHOIS records normally include important lifecycle timestamps:

- **Creation Date** — when the domain was first registered.
- **Updated Date** — the last time the record was modified.
- **Expiration Date** — the date the domain registration will expire if not renewed.

These dates are useful for domain monitoring, brand protection, and expired-domain research.

#### Name Servers (DNS)

WHOIS also lists **Name Servers** responsible for DNS resolution.

Example:

```text
ns1.exampledns.com
ns2.exampledns.com
```

These servers map the domain name to IP addresses and direct traffic to the correct hosting infrastructure.

#### Technical & Administrative Contacts

Some WHOIS records may include contacts responsible for managing the domain:

- **Administrative Contact**
- **Technical Contact**
- **Billing Contact**

Due to privacy regulations such as **GDPR**, these fields are often hidden.

#### IP Address & ASN WHOIS

WHOIS is also used to look up **IP address ownership and network information**. Information may include:

- Network Range
- Organization / ISP
- Country
- ASN (Autonomous System Number)
- Abuse Contact Information

Example use cases: network troubleshooting, abuse reporting, security investigations.

### Why Use WHOIS?

WHOIS is useful for several purposes:

- Identifying the **owner or registrar of a domain**
- Checking **domain availability or expiration dates**
- Performing **cybersecurity investigations**
- Supporting **incident response and threat intelligence**
- Contacting domain owners for **legal or business inquiries**
- Monitoring domains for **trademark protection or brand abuse**

## Examples

### Command-Line WHOIS Lookup

WHOIS can also be queried directly from the terminal.

#### Linux / macOS

Install WHOIS:

```bash
apt install whois
```

Example lookup:

```bash
whois example.com
```

Lookup an IP address:

```bash
whois 8.8.8.8
```

Query a specific WHOIS server:

```bash
whois -h whois.iana.org example.com
```

### WHOIS Lookup Tools

Several online tools allow users to perform WHOIS searches:

| Tool | Description |
|-----|-----|
| https://lookup.icann.org | Official WHOIS lookup provided by ICANN |
| https://whois.domaintools.com | Advanced lookup with historical WHOIS records |
| https://www.whois.com/whois/ | Simple and widely used WHOIS search |
| https://www.godaddy.com/whois | WHOIS lookup provided by GoDaddy |
| https://viewdns.info/whois/ | Additional DNS and WHOIS analysis |

> [!NOTE]
> **Screenshot**
> ![ICANN Lookup web page showing a WHOIS result for example.com with registrar, registrant status, creation/expiry dates, and name servers](placeholder-screenshot)

### RDAP (Modern Replacement for WHOIS)

Many registries are transitioning to **RDAP (Registration Data Access Protocol)**. RDAP provides structured JSON responses, better authentication, rate limiting, and internationalization support.

Example RDAP query:

```bash
curl https://rdap.org/domain/example.com
```

## Security Considerations

### Privacy & Data Protection

Many registrars provide **WHOIS Privacy Protection** services that hide registrant data and replace it with proxy contact details. This helps reduce spam, protect personal information, and comply with regulations such as **GDPR**.

In such cases, the WHOIS record may display:

```text
Registrant: REDACTED FOR PRIVACY
```

or a proxy provider such as:

```text
Domains By Proxy LLC
```

## Troubleshooting

### Limitations of WHOIS

While useful, WHOIS has some limitations:

- Data may be **hidden due to privacy laws**
- WHOIS servers often enforce **rate limiting**
- Information may be **outdated or inaccurate**
- Some networks block **port 43** used by WHOIS

## Summary

WHOIS remains a **fundamental tool for domain and network intelligence**, helping administrators, researchers, and investigators understand ownership, registration details, and infrastructure related to internet resources.

## References

- [ICANN Lookup](https://lookup.icann.org)
- [RFC 9083 — JSON Responses for RDAP](https://www.rfc-editor.org/rfc/rfc9083)
- [RFC 3912 — WHOIS Protocol Specification](https://www.rfc-editor.org/rfc/rfc3912)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Domain-Appraisals](Domain-Appraisals.md) — registration data feeding appraisals — related note
- [Domain-Name-Structure](Domain-Name-Structure.md) — naming context for lookups — related note
- [DNS-Records-and-Their-Types](DNS-Records-and-Their-Types.md) — records revealed alongside WHOIS data — related note
