# Active Directory Domain Services (AD DS)

The centralized directory platform that authenticates and authorizes every user and computer in a Windows domain — and the single richest target in an enterprise attack.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Active Directory Domain Services (AD DS) stores and manages directory data — users, groups, computers, and policy — and services logon and resource-access requests across a domain. This module covers the AD DS building blocks (forests, trees, domains, OUs), the roles that keep a directory consistent and available (FSMO roles, the Global Catalog, replication, sites and services), the two authentication protocols that carry domain credentials (Kerberos and NTLM), and the trust relationships that let separate domains share access. It closes with day-to-day administration through PowerShell and the credential stores an attacker ultimately wants — the local SAM versus the domain `NTDS.dit`.

## Learning Objectives

By the end of this module you will be able to:

- Describe the AD DS logical structure — forest, tree, domain, OU — and where FSMO roles, the Global Catalog, and replication fit
- Explain how Kerberos and NTLM authenticate domain principals and where each is vulnerable
- Administer domain users, groups, and OUs with PowerShell and reason about the SAM vs `NTDS.dit` credential stores

## Topics Covered

This module contains **13 notes**.

| Note | Topic |
| --- | --- |
| [Active-Directory-Domain-Services](Active-Directory-Domain-Services.md) | AD DS overview and architecture |
| [Forest-Tree-and-Domain](Forest-Tree-and-Domain.md) | Forest, tree, and domain logical structure |
| [Organizational-Units-OU](Organizational-Units-OU.md) | Organizational Units (OUs) |
| [FSMO-Roles](FSMO-Roles.md) | Flexible Single Master Operation roles |
| [Global-Catalog](Global-Catalog.md) | Global Catalog server and its role |
| [AD-Replication](AD-Replication.md) | Multi-master replication between DCs |
| [AD-Sites-and-Services](AD-Sites-and-Services.md) | Sites, subnets, and replication topology |
| [Trust-Relationships](Trust-Relationships.md) | Domain and forest trust relationships |
| [Kerberos-Authentication](Kerberos-Authentication.md) | Kerberos authentication protocol |
| [NTLM](NTLM.md) | NTLM authentication and its weaknesses |
| [Managing-Domain-Users-and-Groups-with-PowerShell](Managing-Domain-Users-and-Groups-with-PowerShell.md) | Automating domain user/group management |
| [SAM-vs-NTDS.dit](SAM-vs-NTDS.dit.md) | Local SAM vs domain `NTDS.dit` credential stores |
| [LDAP](LDAP.md) | Lightweight Directory Access Protocol (LDAP) |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, stand up a single-DC lab domain and use the walkthroughs in the notes above as self-paced exercises — promote a DC, create an OU tree, delegate control, and inspect FSMO/GC placement with `netdom query fsmo`.

## Best Practices

- Keep a tiered OU design and delegate administration by OU rather than granting broad Domain Admin membership
- Place at least two DCs per domain and monitor replication health (`repadmin /replsummary`)
- Prefer Kerberos over NTLM and disable NTLMv1; document FSMO role holders for disaster recovery

## Security Considerations

> [!WARNING]
> **AD is the crown jewel**
> Domain compromise almost always means enterprise compromise — protect the directory as the highest-value asset it is.

- Protect the `NTDS.dit` file and DC backups; anyone who reads them owns every domain credential
- Constrain Kerberos delegation and audit for Kerberoasting, AS-REP roasting, and abusable trusts
- Enforce least privilege on privileged groups and monitor replication (DCSync) rights

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Users cannot log on / GPOs not applying at one site | Replication failure — check `repadmin /showrepl` and inter-site links in Sites and Services |
| "Domain controller could not be found" | DNS misconfiguration — clients must resolve DC SRV records; verify DNS points to the DC |

## References

- [Active Directory Domain Services overview (Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [MITRE ATT&CK — Valid Accounts / Domain Accounts (T1078.002)](https://attack.mitre.org/techniques/T1078/002/)
- [The Hacker Recipes — Active Directory](https://www.thehacker.recipes/ad/)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Group Policy Objects (GPO)](../Group-Policy-Objects-GPO/Readme.md) — related module
- [Domain Name System (DNS)](../Domain-Name-System-DNS/Readme.md) — related module
- [Windows Server Management](../Windows-Server-Management/Readme.md) — related module
