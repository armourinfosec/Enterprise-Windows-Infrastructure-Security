# Authentication Methods in Windows

Windows and IIS support several HTTP authentication methods, each tailored to different environments, security needs, and use cases: Basic, Digest, and integrated Windows Authentication (NTLM and Kerberos).

## Overview

When a browser or client requests a protected resource on a Windows/IIS server, the server must prove *who* the caller is before [authorization](Authentication-vs-Authorization.md) decides *what* they may do. The method chosen determines how credentials cross the wire — plaintext-in-Base64, a hashed challenge-response, or a cryptographic ticket — and therefore how exposed those credentials are to interception and replay. This note compares the three families [IIS](Internet-Information-Services(IIS).md) exposes and the security trade-offs of each.

> [!NOTE]
> **Authentication vs authorization**
> These methods only establish **identity**. Restricting *access* to resources is a separate concern — always pair authentication with server-side [authorization](Authentication-vs-Authorization.md) policies.

## 1. Basic Authentication

### How It Works

- The client sends the username and password encoded in Base64 in the HTTP `Authorization` header.

- The header looks like:

```http
Authorization: Basic aHIxOkBybW91cjEyMw==
```

- Decoded, it is simply:

```text
username:password
```

### Security Considerations

- Credentials are only Base64 encoded, **not encrypted**.

- Easily intercepted and readable if transmitted over an unencrypted channel.

- **Requires HTTPS** to secure credentials.

- Mostly suitable for internal or intranet use where secure channels exist.

- **Not recommended for public or internet-facing services without additional protections**.

## 2. Digest Authentication

### How It Works

- Server issues a challenge with parameters such as `nonce` and `realm`.

- Client responds with a hashed value computed from:
username, password, nonce, HTTP method, URI.

- The response hash is sent in the HTTP `Authorization` header.

> Example header (simplified):

```http
Authorization: Digest username="hr1", realm="Digest", nonce="+Upgraded+v184958d0bdaa21c2867ac1ea46aadadd0a5e5b4846208dc014ec730bc94ecbbcac64ccae20b582d0d3bcb982bcf09666e7acf700e83398eb0", uri="/", algorithm=MD5-sess, response="5dcb9150416742f5952b51ff87afdd56", qop=auth, nc=00000001, cnonce="f2f75290496e54d3"
```

### Security Advantages Over Basic

- Passwords **are never sent in plaintext**.

- The `nonce` prevents replay attacks.

- Hashing provides better protection over untrusted networks.

### Security Limitations

- Uses **MD5 hashing**, which is cryptographically weak by modern standards.

- Can still be vulnerable to Man-in-the-Middle (MITM) attacks if not used with TLS.

- **Should always be combined with HTTPS**.

## 3. Windows Authentication (NTLM and Kerberos)

Windows Authentication integrates deeply with Microsoft Active Directory environments and supports:

### NTLM (NT LAN Manager)

- Challenge-response protocol using hashed credentials.

- Common in legacy systems or scenarios where Kerberos is unavailable.

- Provides partial replay protection.

- Does **not** provide mutual authentication.

### Kerberos

- Modern, ticket-based authentication protocol.

- Relies on a **Key Distribution Center (KDC)** within Active Directory.

- Provides **mutual authentication** (both client and server verify identities).

- Requires synchronized client-server clocks.

- Supports Single Sign-On (SSO) seamlessly.

### How Windows Authentication Works (General Flow)

1. Client requests access.

2. Server sends a challenge.

3. Client responds with a token (NTLM hash or Kerberos ticket).

4. Server validates token without the password ever crossing the network.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as IIS Server
    participant KDC as KDC (AD)
    C->>S: Request protected resource
    S-->>C: 401 + WWW-Authenticate challenge
    alt Kerberos
        C->>KDC: Request service ticket
        KDC-->>C: Service ticket
        C->>S: Authorization: Negotiate (ticket)
        S->>S: Validate ticket (mutual auth)
    else NTLM
        C->>S: Authorization: NTLM (hash response)
        S->>KDC: Validate via netlogon
    end
    S-->>C: 200 OK (access granted)
```

> Example NTLM header (Base64-encoded binary blob):

```http
Authorization: NTLM TlRMTVNTUAADAAAAGAAYAFgAAAD8APwAcAAAAAAAAABsAQAABgAGAGwBAAAIAAgAcgEAAAAAAABYAAAABYIIAAAAAAAAAAAA0TJGtZwQTGRND1J9VljfQQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAPNLuFchrF7XG+il/IYfsdUBAQAAAAAAAAaOUfViCNwBxdruGLFFjBsAAAAAAgAMAEEAUgBNAE8AVQBSAAEACABEAEMAMAAxAAQAGABhAHIAbQBvAHUAcgAuAGwAbwBjAGEAbAADACIARABDADAAMQAuAGEAcgBtAG8AdQByAC4AbABvAGMAYQBsAAUAGABhAHIAbQBvAHUAcgAuAGwAbwBjAGEAbAAHAAgABo5R9WII3AEGAAQAAgAAAAoAEAAAAAAAAAAAAAAAAAAAAAAACQAiAEgAVABUAFAALwAxADkAMgAuADEANgA4AC4AMQAuADUAMQAAAAAAAAAAAGgAcgAxAGsAYQBsAGkA
```

*Note*: The string is a complex binary blob, including workstation, domain, and hashed credentials.

### Security Highlights

- Passwords are **never transmitted in cleartext**.

- Supports **Single Sign-On (SSO)** for a seamless user experience.

- Kerberos is considered **more secure than NTLM**.

- Requires TLS/SSL to protect tokens in transit.

## Comparison

| Authentication Method | Password Sent? | Encryption Required | Replay Protection | Single Sign-On (SSO) | Security Level |
| :-- | :-- | :-- | :-- | :-- | :-- |
| **Basic** | Yes (Base64-encoded) | Yes (HTTPS mandatory) | No | No | Low |
| **Digest** | No (hashed) | Recommended (HTTPS) | Yes (nonce-based) | No | Medium |
| **Windows NTLM** | No (hashed) | Recommended (HTTPS) | Partial | Yes | Medium |
| **Windows Kerberos** | No (ticket-based) | Recommended (HTTPS) | Yes | Yes | High |

## Security Considerations

> [!WARNING]
> **Never send credentials over cleartext HTTP**
> Every method here is only as safe as the channel beneath it. Basic leaks credentials outright without TLS; even hashed/ticket methods are exposed to relay and MITM without an encrypted transport.

- **Basic** exposes credentials to anyone on the path unless wrapped in HTTPS.
- **Digest** relies on weak MD5 and is not a substitute for TLS.
- **NTLM** is vulnerable to relay and downgrade attacks — a staple of internal-network compromise (see [NTLM](../Active-Directory-Domain-Services-AD-DS/NTLM.md)).
- **Kerberos** is strongest but depends on precise **time synchronization** and a healthy KDC; misconfigured delegation is itself an attack surface.

## Best Practices

- **Prioritize Kerberos** in Active Directory environments for mutual authentication and SSO.
- Enforce **TLS/SSL** for every method to protect credentials and tokens in transit.
- Maintain accurate **time synchronization (NTP)** across clients and servers for Kerberos.
- Disable Basic (and NTLM where feasible) on internet-facing services; restrict NTLM via policy.
- Always combine authentication with **authorization** policies to fully secure resources.

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Kerberos falls back to NTLM unexpectedly | Missing/incorrect SPN or clock skew > 5 min — register the correct SPN and sync time (NTP) |
| Browser repeatedly prompts for credentials | Site not in the Intranet/Trusted zone for Integrated Auth, or wrong provider order in IIS |
| 401 even with valid credentials | Authentication method disabled in IIS, or authorization rules denying the user despite valid identity |

## References

- [IIS authentication overview (Microsoft Learn)](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/security/authentication/)
- [RFC 7617 — The 'Basic' HTTP Authentication Scheme](https://www.rfc-editor.org/rfc/rfc7617)
- [RFC 7616 — HTTP Digest Access Authentication](https://www.rfc-editor.org/rfc/rfc7616)
- [Kerberos authentication (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)

## Related

- [Authentication-vs-Authorization](Authentication-vs-Authorization.md) — distinguishes auth from access control
- [Internet-Information-Services(IIS)](Internet-Information-Services(IIS).md) — IIS consumes these Windows auth methods
- [NTLM](../Active-Directory-Domain-Services-AD-DS/NTLM.md) — NTLM protocol internals and attacks
- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub
- Web-Application-Penetration-Test — attacking web authentication
