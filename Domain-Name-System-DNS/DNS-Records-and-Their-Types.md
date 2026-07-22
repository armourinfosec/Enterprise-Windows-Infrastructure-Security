# DNS Records and Their Types

DNS records are the individual entries within a DNS zone that direct traffic on the internet — mapping names to addresses, routing email, delegating authority, and carrying verification data. This note breaks down the main record types, shows how each is written in a zone file, and demonstrates querying them with `nslookup`.

## Overview

Every DNS record shares a common form: an **owner name**, a **TTL** (Time to Live, in seconds), an **IN** (Internet) class, a **record type**, and **type-specific data**. Records live inside a zone (see [Forward-and-Reverse-DNS-Zones](Forward-and-Reverse-DNS-Zones.md)), and authoritative name servers serve them to resolvers and clients as queries walk down the [DNS hierarchy](DNS-Hierarchy-and-How-It-Works.md).

The table below summarizes the record types covered in this note:

| Record Type | Name                                  | Purpose                                                        |
| ----------- | ------------------------------------- | ------------------------------------------------------------- |
| **A**       | Address record                        | Maps a domain name to an IPv4 address.                        |
| **AAAA**    | IPv6 address record                   | Maps a domain name to an IPv6 address.                        |
| **CNAME**   | Canonical Name record                 | Creates an alias pointing to another domain name.            |
| **MX**      | Mail Exchange record                  | Specifies the mail server(s) that receive email for a domain. |
| **TXT**     | Text record                           | Stores text data (SPF, DKIM, DMARC, domain verification).    |
| **NS**      | Name Server record                    | Defines the authoritative name servers for a domain.         |
| **SOA**     | Start of Authority record             | Holds administrative/zone-management info for a zone.        |
| **PTR**     | Pointer record                        | Reverse DNS — maps an IP address back to a domain name.      |
| **SRV**     | Service record                        | Locates services (SIP, XMPP, LDAP) by host and port.         |
| **CAA**     | Certification Authority Authorization | Restricts which CAs may issue TLS certificates for a domain. |

> [!NOTE]
> **Common fields**
> In every example below, the owner name ends with a trailing dot (a fully qualified name), `3600` is the TTL in seconds (1 hour), and `IN` is the Internet class — the standard for all public DNS records.

## Record Types

Each record type is defined below with its zone-file syntax, a field-by-field explanation, and a running example zone for **armour.com** that grows as records are added.

### A Record (Address Record)

- Maps a domain name to an IPv4 address.

**Example:**

```text
example.com.  3600  IN  A  192.168.1.100
```

#### Explanation

|Field|Description|
|---|---|
|**example.com.**|The domain or subdomain the record applies to.|
|**3600**|TTL (Time to Live) in seconds (1 hour here).|
|**IN**|Internet class (standard for DNS records).|
|**A**|Record type: Address record mapping domain to IPv4.|
|**192.168.1.100**|IPv4 address the domain resolves to.|

#### DNS Zone Records — armour.com

| Hostname         | Record Type | Value        |
|------------------|-------------|--------------|
| www.armour.com   | A           | 20.20.20.21  |
| mail.armour.com  | A           | 21.20.20.20  |
| ftp.armour.com   | A           | 20.20.20.220 |
| admin.armour.com | A           | 126.25.26.24 |
| ns1.armour.com   | A           | 21.0.0.2     |
| ns2.armour.com   | A           | 21.0.0.3     |

#### Purpose

- Directs browsers or clients to the correct IPv4 address for a domain.

- Essential for routing traffic to web servers or other services.

### AAAA Record (IPv6 Address Record)

- Similar to an A record, but for IPv6 addresses.

**Example:**

```text
example.com.  3600  IN  AAAA  2001:db8::ff00:42:8329
```

#### Explanation

|Field|Description|
|---|---|
|**example.com.**|Domain or subdomain for the record.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**AAAA**|Record type for IPv6 address.|
|**2001:db8::...**|IPv6 address the domain resolves to.|

#### DNS Zone Records — armour.com

| Hostname         | Record Type | Value                         |
|------------------|-------------|--------------------------------|
| www.armour.com   | A           | 20.20.20.21                    |
| www.armour.com   | AAAA        | 2606:50c0:8000::21             |
| mail.armour.com  | A           | 21.20.20.20                    |
| mail.armour.com  | AAAA        | 2606:50c0:8000::22             |
| ftp.armour.com   | A           | 20.20.20.220                   |
| ftp.armour.com   | AAAA        | 2606:50c0:8000::23             |
| admin.armour.com | A           | 126.25.26.24                   |
| admin.armour.com | AAAA        | 2606:50c0:8000::24             |
| ns1.armour.com   | A           | 21.0.0.2                       |
| ns1.armour.com   | AAAA        | 2606:50c0:8000::2              |
| ns2.armour.com   | A           | 21.0.0.3                       |
| ns2.armour.com   | AAAA        | 2606:50c0:8000::3              |

#### Purpose

- Maps a domain to an IPv6 address, enabling access over IPv6 networks.

- Important for future-proofing and compatibility as IPv6 adoption grows.

### CNAME Record (Canonical Name Record)

- Creates an alias for another domain name.

- Useful for pointing multiple subdomains to a single domain.

**Example:**

```text
www.example.com.  3600  IN  CNAME  example.com.
```

```text
blog.example.com.  3600  IN  CNAME  myblog.wordpress.com.
```

#### Explanation

|Field|Description|
|---|---|
|**blog.example.com.**|The domain/subdomain you’re defining.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**CNAME**|Alias record type pointing to another domain.|
|**myblog.wordpress.com.**|Target domain the alias points to.|

#### DNS Zone Records — armour.com

| Hostname         | Record Type | Value                         |
|------------------|-------------|--------------------------------|
| www.armour.com   | A           | 20.20.20.21                    |
| www.armour.com   | AAAA        | 2606:50c0:8000::21             |
| mail.armour.com  | A           | 21.20.20.20                    |
| mail.armour.com  | AAAA        | 2606:50c0:8000::22             |
| ftp.armour.com   | A           | 20.20.20.220                   |
| ftp.armour.com   | AAAA        | 2606:50c0:8000::23             |
| admin.armour.com | A           | 126.25.26.24                   |
| admin.armour.com | AAAA        | 2606:50c0:8000::24             |
| ns1.armour.com   | A           | 21.0.0.2                       |
| ns1.armour.com   | AAAA        | 2606:50c0:8000::2              |
| ns2.armour.com   | A           | 21.0.0.3                       |
| ns2.armour.com   | AAAA        | 2606:50c0:8000::3              |
| webmail.armour.com | CNAME     | mail.armour.com                |
| portal.armour.com  | CNAME     | admin.armour.com               |
| files.armour.com   | CNAME     | ftp.armour.com                 |

#### Purpose

- Allows one domain to act as an alias for another, simplifying management.

- Useful for directing many subdomains to the same place without duplicating records.

### MX Record (Mail Exchange Record)

- Specifies the mail server responsible for receiving emails.

**Example:**

```text
example.com.  3600  IN  MX  10 mail.example.com.
```

#### Explanation

|Field|Description|
|---|---|
|**example.com.**|Domain name this mail record applies to.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**MX**|Mail Exchange record type.|
|**10**|Priority of this mail server (lower = higher).|
|**mail.example.com.**|Hostname of the mail server handling email.|

#### DNS Zone Records — armour.com

| Hostname           | Record Type | Value                         |
|--------------------|-------------|--------------------------------|
| www.armour.com     | A           | 20.20.20.21                    |
| www.armour.com     | AAAA        | 2606:50c0:8000::21             |
| mail.armour.com    | A           | 21.20.20.20                    |
| mail.armour.com    | AAAA        | 2606:50c0:8000::22             |
| ftp.armour.com     | A           | 20.20.20.220                   |
| ftp.armour.com     | AAAA        | 2606:50c0:8000::23             |
| admin.armour.com   | A           | 126.25.26.24                   |
| admin.armour.com   | AAAA        | 2606:50c0:8000::24             |
| ns1.armour.com     | A           | 21.0.0.2                       |
| ns1.armour.com     | AAAA        | 2606:50c0:8000::2              |
| ns2.armour.com     | A           | 21.0.0.3                       |
| ns2.armour.com     | AAAA        | 2606:50c0:8000::3              |
| webmail.armour.com | CNAME       | mail.armour.com                |
| portal.armour.com  | CNAME       | admin.armour.com               |
| files.armour.com   | CNAME       | ftp.armour.com                 |
| armour.com         | MX (10)     | mail.armour.com                |
| armour.com         | MX (20)     | mail2.armour.com               |
| mail2.armour.com   | A           | 20.20.23.23                    |
| mail2.armour.com   | AAAA        | 2606:50c0:8000::25             |

#### Purpose

- Directs email to the appropriate mail servers for a domain.

- Allows prioritization of multiple mail servers for redundancy.

### TXT Record (Text Record)

- Stores text-based data for verification or security purposes.

> [!TIP]
> **Email authentication**
> The three most important TXT records for email security — **SPF**, **DKIM**, and **DMARC** — work together to prevent spoofing and phishing. See the subsections below.

#### SPF (Sender Policy Framework)

```text
example.com.  3600  IN  TXT  "v=spf1 include:_spf.google.com ~all"
```

**Explanation:**

|Field|Description|
|---|---|
|**example.com.**|Domain this TXT record applies to.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**TXT**|Text record type.|
|**"v=spf1 include:_spf.google.com ~all"**|SPF record content specifying authorized mail senders.|

**Purpose:**

- Helps prevent email spoofing by specifying allowed email senders.

- Improves email deliverability and domain reputation.

#### DKIM (DomainKeys Identified Mail)

```text
default._domainkey.example.com.  3600  IN  TXT  "v=DKIM1; p=MIGfMA0..."
```

**Explanation:**

|Field|Description|
|---|---|
|**default._domainkey.example.com.**|Selector (`default`) and domain for DKIM record.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**TXT**|Text record containing DKIM public key.|
|**"v=DKIM1; p=MIGfMA0..."**|DKIM version and base64-encoded public key.|

**Purpose:**

- Adds a cryptographic signature to emails, verifying authenticity and integrity.

- Helps reduce email spoofing and phishing.

#### DMARC (Domain-based Message Authentication, Reporting & Conformance)

```text
_dmarc.example.com.  3600  IN  TXT  "v=DMARC1; p=reject; rua=mailto:dmarc@example.com"
```

**Explanation:**

|Field|Description|
|---|---|
|**_dmarc.example.com.**|DMARC record domain format (`_dmarc.` prefix).|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**TXT**|Text record type.|
|**"v=DMARC1; p=reject; rua=mailto:dmarc@example.com"**|DMARC version, policy, and reporting email address.|

#### DNS Zone Records — armour.com

| Hostname           | Record Type | Value                                                                                      |
|--------------------|-------------|--------------------------------------------------------------------------------------------|
| www.armour.com     | A           | 20.20.20.21                                                                                |
| www.armour.com     | AAAA        | 2606:50c0:8000::21                                                                         |
| mail.armour.com    | A           | 21.20.20.20                                                                                |
| mail.armour.com    | AAAA        | 2606:50c0:8000::22                                                                         |
| ftp.armour.com     | A           | 20.20.20.220                                                                               |
| ftp.armour.com     | AAAA        | 2606:50c0:8000::23                                                                         |
| admin.armour.com   | A           | 126.25.26.24                                                                               |
| admin.armour.com   | AAAA        | 2606:50c0:8000::24                                                                         |
| ns1.armour.com     | A           | 21.0.0.2                                                                                   |
| ns1.armour.com     | AAAA        | 2606:50c0:8000::2                                                                          |
| ns2.armour.com     | A           | 21.0.0.3                                                                                   |
| ns2.armour.com     | AAAA        | 2606:50c0:8000::3                                                                          |
| webmail.armour.com | CNAME       | mail.armour.com                                                                            |
| portal.armour.com  | CNAME       | admin.armour.com                                                                           |
| files.armour.com   | CNAME       | ftp.armour.com                                                                             |
| armour.com         | MX (10)     | mail.armour.com                                                                            |
| armour.com         | MX (20)     | mail2.armour.com                                                                           |
| mail2.armour.com   | A           | 20.20.23.23                                                                                |
| mail2.armour.com   | AAAA        | 2606:50c0:8000::25                                                                         |
| armour.com         | TXT (SPF)   | "v=spf1 ip4:21.20.20.20 ip4:20.20.23.23 include:_spf.google.com -all"                      |
| default._domainkey.armour.com | TXT (DKIM) | "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A..." |
| _dmarc.armour.com  | TXT (DMARC) | "v=DMARC1; p=quarantine; rua=mailto:dmarc@armour.com; ruf=mailto:dmarc@armour.com; fo=1"  |

**Purpose:**

- Specifies how email receivers should handle unauthenticated messages (reject, quarantine, none).

- Provides reports to domain owners about email authentication activity.

- Protects domain reputation from spoofing and phishing.

### NS Record (Name Server Record)

- Defines the authoritative name servers for a domain.

**Example:**

```text
example.com.  3600  IN  NS  ns1.example.com.
example.com.  3600  IN  NS  ns2.example.com.
```

#### Explanation

|Field|Description|
|---|---|
|**example.com.**|Domain this record applies to.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**NS**|Name Server record type.|
|**ns1.example.com.**|Authoritative name server for the domain.|
|**ns2.example.com.**|Secondary authoritative name server.|

#### DNS Zone Records — armour.com

| Hostname                     | Record Type | Value                                                                                      |
|-------------------------------|-------------|--------------------------------------------------------------------------------------------|
| armour.com                    | NS          | ns1.armour.com                                                                             |
| armour.com                    | NS          | ns2.armour.com                                                                             |
| ns1.armour.com                | A           | 21.0.0.2                                                                                   |
| ns1.armour.com                | AAAA        | 2606:50c0:8000::2                                                                          |
| ns2.armour.com                | A           | 21.0.0.3                                                                                   |
| ns2.armour.com                | AAAA        | 2606:50c0:8000::3                                                                          |
| www.armour.com                | A           | 20.20.20.21                                                                                |
| www.armour.com                | AAAA        | 2606:50c0:8000::21                                                                         |
| mail.armour.com               | A           | 21.20.20.20                                                                                |
| mail.armour.com               | AAAA        | 2606:50c0:8000::22                                                                         |
| ftp.armour.com                | A           | 20.20.20.220                                                                               |
| ftp.armour.com                | AAAA        | 2606:50c0:8000::23                                                                         |
| admin.armour.com              | A           | 126.25.26.24                                                                               |
| admin.armour.com              | AAAA        | 2606:50c0:8000::24                                                                         |
| mail2.armour.com              | A           | 20.20.23.23                                                                                |
| mail2.armour.com              | AAAA        | 2606:50c0:8000::25                                                                         |
| webmail.armour.com            | CNAME       | mail.armour.com                                                                            |
| portal.armour.com             | CNAME       | admin.armour.com                                                                           |
| files.armour.com              | CNAME       | ftp.armour.com                                                                             |
| armour.com                    | MX (10)     | mail.armour.com                                                                            |
| armour.com                    | MX (20)     | mail2.armour.com                                                                           |
| armour.com                    | TXT (SPF)   | "v=spf1 ip4:21.20.20.20 ip4:20.20.23.23 include:_spf.google.com -all"                      |
| default._domainkey.armour.com | TXT (DKIM)  | "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A..."                                   |
| _dmarc.armour.com             | TXT (DMARC) | "v=DMARC1; p=quarantine; rua=mailto:dmarc@armour.com; ruf=mailto:dmarc@armour.com; fo=1"  |

#### Purpose

- Specifies which DNS servers are authoritative for a domain.

- Authoritative servers provide actual DNS data to resolvers and clients.

### SOA Record (Start of Authority)

- Contains administrative information about the domain’s DNS zone.

**Example:**

```text
example.com.  3600  IN  SOA  ns1.example.com. admin.example.com. (
                                2024021101  ; Serial
                                3600        ; Refresh
                                1800        ; Retry
                                1209600     ; Expire
                                86400 )     ; Minimum TTL
```

#### Explanation

|Field|Description|
|---|---|
|**example.com.**|Domain for which the SOA record applies.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**SOA**|Start of Authority record type.|
|**ns1.example.com.**|Primary/master DNS server for the domain.|
|**admin.example.com.**|Email of domain administrator (dot replaces @).|
|**2024021101**|Serial number, version of this zone file.|
|**3600**|Refresh interval: how often secondaries check for updates.|
|**1800**|Retry interval if refresh fails.|
|**1209600**|Expire time: when secondary stops serving zone if no contact.|
|**86400**|Minimum TTL for negative caching.|

#### DNS Zone Records — armour.com

| Hostname                     | Record Type | Value                                                                                      |
|-------------------------------|-------------|--------------------------------------------------------------------------------------------|
| armour.com                    | SOA         | ns1.armour.com. admin.armour.com. (2025100901 7200 3600 1209600 3600)                      |
| armour.com                    | NS          | ns1.armour.com                                                                             |
| armour.com                    | NS          | ns2.armour.com                                                                             |
| ns1.armour.com                | A           | 21.0.0.2                                                                                   |
| ns1.armour.com                | AAAA        | 2606:50c0:8000::2                                                                          |
| ns2.armour.com                | A           | 21.0.0.3                                                                                   |
| ns2.armour.com                | AAAA        | 2606:50c0:8000::3                                                                          |
| www.armour.com                | A           | 20.20.20.21                                                                                |
| www.armour.com                | AAAA        | 2606:50c0:8000::21                                                                         |
| mail.armour.com               | A           | 21.20.20.20                                                                                |
| mail.armour.com               | AAAA        | 2606:50c0:8000::22                                                                         |
| ftp.armour.com                | A           | 20.20.20.220                                                                               |
| ftp.armour.com                | AAAA        | 2606:50c0:8000::23                                                                         |
| admin.armour.com              | A           | 126.25.26.24                                                                               |
| admin.armour.com              | AAAA        | 2606:50c0:8000::24                                                                         |
| mail2.armour.com              | A           | 20.20.23.23                                                                                |
| mail2.armour.com              | AAAA        | 2606:50c0:8000::25                                                                         |
| webmail.armour.com            | CNAME       | mail.armour.com                                                                            |
| portal.armour.com             | CNAME       | admin.armour.com                                                                           |
| files.armour.com              | CNAME       | ftp.armour.com                                                                             |
| armour.com                    | MX (10)     | mail.armour.com                                                                            |
| armour.com                    | MX (20)     | mail2.armour.com                                                                           |
| armour.com                    | TXT (SPF)   | "v=spf1 ip4:21.20.20.20 ip4:20.20.23.23 include:_spf.google.com -all"                      |
| default._domainkey.armour.com | TXT (DKIM)  | "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A..."                                   |
| _dmarc.armour.com             | TXT (DMARC) | "v=DMARC1; p=quarantine; rua=mailto:dmarc@armour.com; ruf=mailto:dmarc@armour.com; fo=1"  |

#### Purpose

- Provides essential administrative info for zone management.

- Controls DNS zone transfers and caching behavior.

### PTR Record (Pointer Record)

- Used for reverse DNS lookups (IP → Domain).

**Example:**

```text
100.1.168.192.in-addr.arpa.  3600  IN  PTR  example.com.
```

#### Explanation

|Field|Description|
|---|---|
|**100.1.168.192.in-addr.arpa.**|Reverse DNS domain for the IP address.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**PTR**|Pointer record type.|
|**example.com.**|Domain name associated with the IP address.|

#### DNS Zone Records — armour.com

| Hostname / IP Address          | Record Type | Value / Hostname                                                                       |
|--------------------------------|-------------|----------------------------------------------------------------------------------------|
| armour.com                     | SOA         | ns1.armour.com. admin.armour.com. (2025100901 7200 3600 1209600 3600)                  |
| armour.com                     | NS          | ns1.armour.com                                                                         |
| armour.com                     | NS          | ns2.armour.com                                                                         |
| ns1.armour.com                 | A           | 21.0.0.2                                                                               |
| ns1.armour.com                 | AAAA        | 2606:50c0:8000::2                                                                      |
| ns2.armour.com                 | A           | 21.0.0.3                                                                               |
| ns2.armour.com                 | AAAA        | 2606:50c0:8000::3                                                                      |
| www.armour.com                 | A           | 20.20.20.21                                                                            |
| www.armour.com                 | AAAA        | 2606:50c0:8000::21                                                                     |
| mail.armour.com                | A           | 21.20.20.20                                                                            |
| mail.armour.com                | AAAA        | 2606:50c0:8000::22                                                                     |
| ftp.armour.com                 | A           | 20.20.20.220                                                                           |
| ftp.armour.com                 | AAAA        | 2606:50c0:8000::23                                                                     |
| admin.armour.com               | A           | 126.25.26.24                                                                           |
| admin.armour.com               | AAAA        | 2606:50c0:8000::24                                                                     |
| mail2.armour.com               | A           | 20.20.23.23                                                                            |
| mail2.armour.com               | AAAA        | 2606:50c0:8000::25                                                                     |
| webmail.armour.com             | CNAME       | mail.armour.com                                                                        |
| portal.armour.com              | CNAME       | admin.armour.com                                                                       |
| files.armour.com               | CNAME       | ftp.armour.com                                                                         |
| armour.com                     | MX (10)     | mail.armour.com                                                                        |
| armour.com                     | MX (20)     | mail2.armour.com                                                                       |
| armour.com                     | TXT (SPF)   | "v=spf1 ip4:21.20.20.20 ip4:20.20.23.23 include:_spf.google.com -all"                  |
| default._domainkey.armour.com  | TXT (DKIM)  | "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A..."                               |
| _dmarc.armour.com              | TXT (DMARC) | "v=DMARC1; p=quarantine; rua=mailto:dmarc@armour.com; ruf=mailto:dmarc@armour.com"     |
| **20.20.20.21.in-addr.arpa**   | PTR         | www.armour.com.                                                                        |
| **21.20.20.20.in-addr.arpa**   | PTR         | mail.armour.com.                                                                       |
| **20.20.20.220.in-addr.arpa**  | PTR         | ftp.armour.com.                                                                        |
| **126.25.26.24.in-addr.arpa**  | PTR         | admin.armour.com.                                                                      |
| **20.20.23.23.in-addr.arpa**   | PTR         | mail2.armour.com.                                                                      |

#### Purpose

- Maps IP addresses back to domain names (reverse DNS).

- Useful for spam filtering, logging, and diagnostics.

### SRV Record (Service Record)

- Specifies the location of services (used in VoIP, XMPP, etc.).

**Example:**

```text
_sip._tcp.example.com.  3600  IN  SRV  10 50 5060 sipserver.example.com.
```

- `10` → Priority

- `50` → Weight

- `5060` → Port

#### Explanation

|Field|Description|
|---|---|
|**_sip._tcp.example.com.**|Service and protocol with domain.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**SRV**|Service record type.|
|**10**|Priority of the target host (lower = higher).|
|**50**|Relative weight for entries with the same priority.|
|**5060**|TCP or UDP port on which the service runs.|
|**sipserver.example.com.**|Target host providing the service.|

#### DNS Zone Records — armour.com

| Hostname / IP Address          | Record Type | Value / Hostname                                                                       |
|--------------------------------|-------------|----------------------------------------------------------------------------------------|
| armour.com                     | SOA         | ns1.armour.com. admin.armour.com. (2025100901 7200 3600 1209600 3600)                  |
| armour.com                     | NS          | ns1.armour.com                                                                         |
| armour.com                     | NS          | ns2.armour.com                                                                         |
| ns1.armour.com                 | A           | 21.0.0.2                                                                               |
| ns1.armour.com                 | AAAA        | 2606:50c0:8000::2                                                                      |
| ns2.armour.com                 | A           | 21.0.0.3                                                                               |
| ns2.armour.com                 | AAAA        | 2606:50c0:8000::3                                                                      |
| www.armour.com                 | A           | 20.20.20.21                                                                            |
| www.armour.com                 | AAAA        | 2606:50c0:8000::21                                                                     |
| mail.armour.com                | A           | 21.20.20.20                                                                            |
| mail.armour.com                | AAAA        | 2606:50c0:8000::22                                                                     |
| ftp.armour.com                 | A           | 20.20.20.220                                                                           |
| ftp.armour.com                 | AAAA        | 2606:50c0:8000::23                                                                     |
| admin.armour.com               | A           | 126.25.26.24                                                                           |
| admin.armour.com               | AAAA        | 2606:50c0:8000::24                                                                     |
| mail2.armour.com               | A           | 20.20.23.23                                                                            |
| mail2.armour.com               | AAAA        | 2606:50c0:8000::25                                                                     |
| webmail.armour.com             | CNAME       | mail.armour.com                                                                        |
| portal.armour.com              | CNAME       | admin.armour.com                                                                       |
| files.armour.com               | CNAME       | ftp.armour.com                                                                         |
| armour.com                     | MX (10)     | mail.armour.com                                                                        |
| armour.com                     | MX (20)     | mail2.armour.com                                                                       |
| armour.com                     | TXT (SPF)   | "v=spf1 ip4:21.20.20.20 ip4:20.20.23.23 include:_spf.google.com -all"                  |
| default._domainkey.armour.com  | TXT (DKIM)  | "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A..."                               |
| _dmarc.armour.com              | TXT (DMARC) | "v=DMARC1; p=quarantine; rua=mailto:dmarc@armour.com; ruf=mailto:dmarc@armour.com"     |
| 20.20.20.21.in-addr.arpa       | PTR         | www.armour.com.                                                                        |
| 21.20.20.20.in-addr.arpa       | PTR         | mail.armour.com.                                                                       |
| 20.20.20.220.in-addr.arpa      | PTR         | ftp.armour.com.                                                                        |
| 126.25.26.24.in-addr.arpa      | PTR         | admin.armour.com.                                                                      |
| 20.20.23.23.in-addr.arpa       | PTR         | mail2.armour.com.                                                                      |
| _sip._tcp.armour.com           | SRV         | 10 60 5060 sipserver.armour.com.                                                      |
| _xmpp-client._tcp.armour.com  | SRV         | 5 30 5222 xmpp.armour.com.                                                            |
| _ldap._tcp.armour.com          | SRV         | 0 100 389 ldap.armour.com.                                                            |

#### Purpose

- Directs clients to specific servers and ports for particular services.

- Enables load balancing and failover for services like SIP, XMPP.

### CAA Record (Certification Authority Authorization)

- Restricts which Certificate Authorities (CAs) can issue SSL certificates for a domain.

**Example:**

```text
example.com.  3600  IN  CAA  0 issue "letsencrypt.org"
```

#### Explanation

|Field|Description|
|---|---|
|**example.com.**|Domain name this CAA record applies to.|
|**3600**|TTL in seconds.|
|**IN**|Internet class.|
|**CAA**|Certification Authority Authorization record type.|
|**0**|Flag (usually 0).|
|**issue**|Property tag, specifying allowed CAs to issue certs.|
|**"letsencrypt.org"**|The authorized CA domain name.|

#### DNS Zone Records — armour.com

| Hostname / IP Address           | Record Type | Value / Hostname                                                                       |
|---------------------------------|-------------|----------------------------------------------------------------------------------------|
| armour.com                      | SOA         | ns1.armour.com. admin.armour.com. (2025100901 7200 3600 1209600 3600)                  |
| armour.com                      | NS          | ns1.armour.com                                                                         |
| armour.com                      | NS          | ns2.armour.com                                                                         |
| ns1.armour.com                  | A           | 21.0.0.2                                                                               |
| ns1.armour.com                  | AAAA        | 2606:50c0:8000::2                                                                      |
| ns2.armour.com                  | A           | 21.0.0.3                                                                               |
| ns2.armour.com                  | AAAA        | 2606:50c0:8000::3                                                                      |
| www.armour.com                  | A           | 20.20.20.21                                                                            |
| www.armour.com                  | AAAA        | 2606:50c0:8000::21                                                                     |
| mail.armour.com                 | A           | 21.20.20.20                                                                            |
| mail.armour.com                 | AAAA        | 2606:50c0:8000::22                                                                     |
| ftp.armour.com                  | A           | 20.20.20.220                                                                           |
| ftp.armour.com                  | AAAA        | 2606:50c0:8000::23                                                                     |
| admin.armour.com                | A           | 126.25.26.24                                                                           |
| admin.armour.com                | AAAA        | 2606:50c0:8000::24                                                                     |
| mail2.armour.com                | A           | 20.20.23.23                                                                            |
| mail2.armour.com                | AAAA        | 2606:50c0:8000::25                                                                     |
| webmail.armour.com              | CNAME       | mail.armour.com                                                                        |
| portal.armour.com               | CNAME       | admin.armour.com                                                                       |
| files.armour.com                | CNAME       | ftp.armour.com                                                                         |
| armour.com                      | MX (10)     | mail.armour.com                                                                        |
| armour.com                      | MX (20)     | mail2.armour.com                                                                       |
| armour.com                      | TXT (SPF)   | "v=spf1 ip4:21.20.20.20 ip4:20.20.23.23 include:_spf.google.com -all"                  |
| default._domainkey.armour.com   | TXT (DKIM)  | "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A..."                               |
| _dmarc.armour.com               | TXT (DMARC) | "v=DMARC1; p=quarantine; rua=mailto:dmarc@armour.com; ruf=mailto:dmarc@armour.com"     |
| 20.20.20.21.in-addr.arpa        | PTR         | www.armour.com.                                                                        |
| 21.20.20.20.in-addr.arpa        | PTR         | mail.armour.com.                                                                       |
| 20.20.20.220.in-addr.arpa       | PTR         | ftp.armour.com.                                                                        |
| 126.25.26.24.in-addr.arpa       | PTR         | admin.armour.com.                                                                      |
| 20.20.23.23.in-addr.arpa        | PTR         | mail2.armour.com.                                                                      |
| _sip._tcp.armour.com            | SRV         | 10 60 5060 sipserver.armour.com.                                                      |
| _xmpp-client._tcp.armour.com   | SRV         | 5 30 5222 xmpp.armour.com.                                                            |
| _ldap._tcp.armour.com           | SRV         | 0 100 389 ldap.armour.com.                                                            |
| armour.com                      | CAA (0 issue)       | "letsencrypt.org"                                                                      |
| armour.com                      | CAA (0 issuewild)   | "letsencrypt.org"                                                                      |
| armour.com                      | CAA (0 iodef)       | "mailto:security@armour.com"                                                          |

#### Purpose

- Improves SSL/TLS security by specifying which CAs can issue certificates for your domain.

- Helps prevent unauthorized certificate issuance.

## Administration

### Using nslookup for DNS Queries

`nslookup` is a command-line tool used for querying DNS records. Here’s a rundown of the commands:

#### Basic Query

- Performs a default DNS lookup for the A record (IPv4) of `armourinfosec.com`.

```cmd
nslookup armourinfosec.com
```

#### A Record Lookup

- Explicitly queries for the A record of `armourinfosec.com`.

```cmd
nslookup -type=A armourinfosec.com
```

#### A Record for Another Domain

- Looks up the A record for `www.armourinfosec.com`.

```cmd
nslookup -type=A www.armourinfosec.com
```

#### AAAA Record (IPv6)

- Retrieves the IPv6 (AAAA) record for `armourinfosec.com`.

```cmd
nslookup -type=AAAA armourinfosec.com
```

#### MX Record Lookup

- Fetches the mail exchange (MX) records for `armourinfosec.com`.

```cmd
nslookup -type=MX armourinfosec.com
```

#### TXT Record Lookup

- Queries for any TXT records associated with `armourinfosec.com`.

    - Typically used for SPF, DKIM, or DMARC configurations.

```cmd
nslookup -type=txt armourinfosec.com
```

#### NS Record Lookup

- Retrieves the authoritative name servers (NS) for `armourinfosec.com`.

```cmd
nslookup -type=ns armourinfosec.com
```

#### SOA Record Lookup

- Retrieves the Start of Authority (SOA) record for `armourinfosec.com`.

```cmd
nslookup -type=SOA armourinfosec.com
```

#### Reverse DNS Lookup

- Performs a reverse DNS lookup of the IP `8.8.8.8` to find the domain name associated with it.

```cmd
nslookup 8.8.8.8
```

#### Querying a Specific DNS Server

- Queries `armourinfosec.com` using the DNS server at `64.6.64.6`.

```cmd
nslookup armourinfosec.com 64.6.64.6
```

#### Launching Interactive Mode

- nslookup enters interactive mode, indicated by the `>` prompt.

```cmd
nslookup
```

> [!NOTE]
> **Screenshot**
> ![nslookup running in interactive mode, showing the ">" prompt and the default server and address lines returned after launching the tool](placeholder-screenshot)

#### Commands in Interactive Mode

1. **Query default domain**

- Typing a domain name directly will query the default DNS server for its A record.

```text
google.com
```

```text
facebook.com
```

2. **Change Query Type**

- Sets the query type to **NS** to retrieve name server records.

```text
set type=NS
```

- Sets the query type to **MX** to retrieve mail exchange records.

```text
set type=MX
```

3. **Change DNS Server**

- Switches the DNS server used by `nslookup` to **64.6.64.6** (an alternate resolver like Verisign’s Public DNS).

 - Once set, all future queries use this DNS server.

```text
server 64.6.64.6
```

### DNS Cache Management

#### View Cached DNS Entries

- Displays the current contents of the DNS resolver cache on your system.
- Helps in troubleshooting name resolution issues by showing what entries are cached locally.

```cmd
ipconfig /displaydns
```

#### Clear the DNS Resolver Cache

- Clears the DNS cache, forcing the system to resolve hostnames from scratch.
- Useful if DNS entries have changed, or you’re experiencing stale/broken resolutions.

```cmd
ipconfig /flushdns
```

## Security Considerations

DNS records are a reconnaissance goldmine: they map an organization's internal and external infrastructure, and several record types leak far more than intended. From the attacker's perspective, enumerating records is often the first step of an engagement; from the defender's, controlling *what* is resolvable and *by whom* limits that exposure.

> [!WARNING]
> **Records leak infrastructure**
> - **Zone transfers (AXFR)** — a misconfigured authoritative server that allows unrestricted AXFR hands an attacker the **entire zone**: every A, CNAME, MX, SRV, and TXT record at once. Restrict transfers to named secondary servers only.
> - **SRV records expose Active Directory** — locator records such as `_ldap._tcp.dc._msdcs`, `_kerberos._tcp`, and `_gc._tcp` reveal Domain Controllers, Kerberos KDCs, and Global Catalog servers, letting an attacker map the AD topology with a single query.
> - **TXT records reveal third parties** — SPF `include:` mechanisms and domain-verification tokens disclose which SaaS/email providers a domain uses, widening the supply-chain attack surface.
> - **Dangling CNAME / subdomain takeover** — a CNAME pointing to a de-provisioned cloud resource can be re-registered by an attacker, hijacking the subdomain.
> - **PTR and reverse zones** — reverse lookups map IP ranges back to hostnames, exposing internal naming schemes and live hosts.

Attempting a zone transfer is a standard recon check — it should fail against a hardened server:

```bash
dig AXFR armour.com @ns1.armour.com
```

Locating AD Domain Controllers via SRV records:

```cmd
nslookup -type=SRV _ldap._tcp.dc._msdcs.armour.local
```

Defensively, DNSSEC (see [DNSSEC](DNSSEC.md)) protects record *integrity* against spoofing and cache poisoning but does **not** provide confidentiality — signed records are still public. Split-horizon DNS (see [Split-DNS](Split-DNS.md)) is what keeps internal names off the public internet.

## Best Practices

- **Restrict zone transfers** to explicitly named secondary servers; never allow AXFR to `any`.
- **Set TTLs deliberately** — short (300–900 s) for records that change or during migrations, longer (3600 s+) for stable records to cut query load.
- **Publish SPF, DKIM, and DMARC** TXT records together, and use a strict DMARC policy (`p=reject` or `p=quarantine`) to stop spoofing.
- **Add CAA records** to constrain which CAs may issue certificates for the domain, reducing mis-issuance risk.
- **Audit for dangling records** — remove CNAME/A entries pointing to decommissioned resources to prevent subdomain takeover.
- **Keep the SOA serial incrementing** on every zone edit so secondaries pick up changes reliably.

## Troubleshooting

| Symptom | Likely cause & fix |
|---------|--------------------|
| Record change not visible to clients | Stale cache — wait out the TTL, or run `ipconfig /flushdns` locally and clear the server cache. |
| Secondary server serving old data | SOA **serial** was not incremented after the edit, or zone transfer failed — bump the serial and check AXFR/IXFR. |
| `nslookup` returns the wrong / no record type | Default query is an A record — set the type explicitly, e.g. `nslookup -type=MX armour.com`. |
| Email rejected or marked spam | Missing or malformed SPF/DKIM/DMARC TXT records — validate syntax and alignment. |
| CNAME "no answer" for the apex/root | CNAME is not allowed at a zone apex — use an A/AAAA record (or ALIAS/ANAME provider feature) instead. |
| Reverse lookup fails | Missing PTR record or reverse (`in-addr.arpa` / `ip6.arpa`) zone not delegated to you by the IP owner. |

## References

- RFC 1035 — Domain Names, Implementation and Specification (A, NS, SOA, CNAME, MX, PTR, TXT): https://www.rfc-editor.org/rfc/rfc1035
- RFC 2782 — A DNS RR for specifying the location of services (SRV): https://www.rfc-editor.org/rfc/rfc2782
- RFC 8659 — DNS Certification Authority Authorization (CAA) Resource Record: https://www.rfc-editor.org/rfc/rfc8659
- IANA — DNS Parameters (Resource Record TYPEs registry): https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml

## Related
- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [DNS-Hierarchy-and-How-It-Works](DNS-Hierarchy-and-How-It-Works.md) — how records resolve through the hierarchy — related note
- [Forward-and-Reverse-DNS-Zones](Forward-and-Reverse-DNS-Zones.md) — zones that store these records — related note
- [DNS-Server-Types](DNS-Server-Types.md) — servers that serve these records — related note
- [DNS-Cache](DNS-Cache.md) — client-side caching of resolved records — related note
- [DNSSEC](DNSSEC.md) — cryptographically signing these records — related note
- [Split-DNS](Split-DNS.md) — internal vs external record views — related note
- [Whois](Whois.md) — registration data that complements DNS recon — related note
