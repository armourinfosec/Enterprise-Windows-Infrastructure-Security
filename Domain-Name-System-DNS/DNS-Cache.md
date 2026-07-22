# DNS Cache

The **DNS Cache** (DNS Resolver Cache) is a temporary, client-side store of recently resolved DNS lookups kept by Windows so repeat requests can be answered locally instead of querying a DNS server every time.

## Overview

When you visit a website like `example.com`, Windows performs a DNS lookup to determine its IP address. Instead of performing the lookup every time, Windows stores the result in the **DNS Resolver Cache**.

This improves:

- Performance
- Network efficiency
- Browsing speed
- Faster domain resolution

> [!NOTE]
> **Client cache vs server cache**
> This note covers the **client-side** resolver cache managed by the Windows DNS Client service. The equivalent store on a DNS server is the [DNS-Server-Cache](DNS-Server-Cache.md).

## Concepts

### Example Cache Entry

```text
Windows IP Configuration

    example.com
    ----------------------------------------
    Record Name . . . . . : example.com
    Record Type . . . . . : 1
    Time To Live  . . . . : 300
    Data Length . . . . . : 4
    Section . . . . . . . : Answer
    A (Host) Record . . . : 93.184.216.34
```

### Explanation of Fields

| Field | Description |
| --- | --- |
| Record Name | The domain name |
| Record Type | DNS record type |
| Time To Live (TTL) | Time remaining before cache expires |
| Data Length | Size of the DNS record |
| Section | DNS response section |
| A Record | The resolved IPv4 address |

### Common DNS Record Types

| Type | Meaning |
| --- | --- |
| A | IPv4 address |
| AAAA | IPv6 address |
| CNAME | Canonical name (alias) |
| MX | Mail server |
| NS | Name server |
| PTR | Reverse DNS |

> [!NOTE]
> **Record types in depth**
> See [DNS-Records-and-Their-Types](DNS-Records-and-Their-Types.md) for the full catalog of record types and their formats.

## Administration

### Display the DNS Cache

Check which domains were recently resolved.

```cmd
ipconfig /displaydns
```

> [!NOTE]
> **Screenshot**
> ![ipconfig /displaydns output in a Command Prompt window listing cached record names, record types, TTL, and resolved A records](placeholder-screenshot)

### Flush the DNS Cache

Clears all cached DNS entries.

```cmd
ipconfig /flushdns
```

### Register DNS

Refresh DNS records with the DHCP server.

```cmd
ipconfig /registerdns
```

### View Full Network Configuration

```cmd
ipconfig /all
```

### Filtering DNS Cache Results

Search for a specific domain:

```cmd
ipconfig /displaydns | findstr google
```

Search for a specific keyword:

```cmd
ipconfig /displaydns | findstr cloud
```

## Security Considerations

The DNS resolver cache is a valuable source of host-based intelligence and a target for tampering.

### 1. Investigating Visited Domains

Check which domains were recently resolved. Useful during **forensics or incident response**.

```cmd
ipconfig /displaydns
```

### 2. Detecting Malware Activity

Malware often communicates with hidden domains. Cached DNS entries may reveal suspicious or unknown domains.

### 3. Troubleshooting DNS Issues

Helps verify if a domain is resolving to the correct IP address.

### 4. Identifying Internal Infrastructure

Cached DNS entries may reveal:

- Internal services
- Development servers
- Hidden subdomains
- API endpoints

> [!WARNING]
> **Cache poisoning**
> Flushing the DNS cache can resolve **DNS poisoning or outdated records**. If a host resolves a known-good domain to an unexpected IP, treat the cache as suspect and flush it.

## Best Practices

- DNS cache entries expire based on their **TTL (Time To Live)**.
- Cached entries are automatically removed after expiration.
- Flush the cache when troubleshooting stale or poisoned records.

## Troubleshooting

| Command | Purpose |
| --- | --- |
| `ipconfig /displaydns` | Display DNS resolver cache |
| `ipconfig /flushdns` | Clear DNS cache |
| `ipconfig /registerdns` | Refresh DNS registration |
| `ipconfig /all` | Show full network configuration |

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [DNS-Server-Cache](DNS-Server-Cache.md) — server-side cache counterpart — related note
- [Recursive-(Caching)-DNS-Server](Recursive-(Caching)-DNS-Server.md) — resolver that populates the cache — related note
- [DNS-Hierarchy-and-How-It-Works](DNS-Hierarchy-and-How-It-Works.md) — how lookups fill the cache — related note
- [DNS-Records-and-Their-Types](DNS-Records-and-Their-Types.md) — record types stored in the cache — related note
