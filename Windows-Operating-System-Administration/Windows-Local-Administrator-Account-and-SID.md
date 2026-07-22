# Windows Local Administrator Account and SID

Every Windows installation ships with a built-in local **Administrator** account, and every security principal on the system is tracked internally not by name but by a **Security Identifier (SID)**. This note breaks down the structure of a SID, the well-known **Relative Identifier (RID) 500** that marks the built-in Administrator, and the commands used to enumerate accounts and their SIDs.

## Overview

Windows authorizes access by SID, never by account name. A SID is a variable-length, immutable value assigned when a principal (user, group, or computer) is created; access control entries, tokens, and logs all reference the SID rather than the display name. Because the SID is fixed for the life of the account, **renaming the Administrator account does not change its identity** — it still ends in the well-known RID `500`.

Local accounts live in the [SAM](../Active-Directory-Domain-Services-AD-DS/SAM-vs-NTDS.dit.md) database, so a local principal's SID is built from the **machine SID** plus a RID. Domain principals draw their SID from the domain SID instead. Understanding this split is what lets you tell a local `SRV01\Administrator` from a domain `CONTOSO\Administrator` at a glance. Related account-management workflows are covered in [User-Management](User-Management.md), [PowerShell-User-Group-Management](PowerShell-User-Group-Management.md), and [Add-User-to-Administrators](Add-User-to-Administrators.md).

## SID Structure

A SID is a sequence of fields joined by hyphens. Taking the built-in Administrator SID from a standalone server as the worked example:

```text
S-1-5-21-3574371106-2527126871-2867751051-500
│ │ │  └──────────────┬──────────────┘  └─ Relative Identifier (RID)
│ │ │      Unique Machine / Domain Identifier
│ │ └── Sub-authority: 21 = a machine or AD domain SID
│ └──── Identifier Authority: 5 = NT Authority
└────── SID Revision (always 1)
```

### Components

| Component | Description |
|-----------|-------------|
| `S` | Literal prefix indicating a Security Identifier |
| `1` | SID revision level (always 1) |
| `5` | Identifier authority — `5` is **NT Authority** |
| `21` | Sub-authority marking a unique machine or Active Directory domain SID |
| `3574371106-2527126871-2867751051` | Unique machine / domain identifier |
| `500` | Relative Identifier (RID) — identifies the principal within that machine/domain |

## Machine SID

The **machine SID** is the full SID with the trailing RID removed:

```text
S-1-5-21-3574371106-2527126871-2867751051
```

Every local principal on the host shares this machine SID; the RID is appended to it to form each account's complete SID. The Administrator's full SID is therefore the machine SID plus `-500`.

> [!NOTE]
> **The SID outlives the name**
> An account can be renamed, and its display name is cosmetic. The SID — and specifically the RID suffix — stays constant for the life of the account. This is why the RID `500` account is a reliable way to locate the true built-in Administrator regardless of what it has been renamed to.

## Relative Identifier (RID)

The **RID** is the last sub-field of a SID and uniquely identifies a principal within its machine or domain scope. For the example account:

```text
RID = 500
```

RID **500** is reserved for the built-in **Administrator** account on every Windows system.

### Common Well-Known RIDs

| RID | Account / Group |
|-----:|-----------------|
| 500 | Built-in Administrator |
| 501 | Guest |
| 502 | KRBTGT (Domain Controllers only) |
| 512 | Domain Admins |
| 513 | Domain Users |
| 514 | Domain Guests |
| 515 | Domain Computers |
| 516 | Domain Controllers |
| 518 | Schema Admins |
| 519 | Enterprise Admins |
| 520 | Group Policy Creator Owners |
| 544 | Administrators (built-in local group) |
| 545 | Users |
| 546 | Guests |
| 547 | Power Users |
| 548 | Account Operators |
| 549 | Server Operators |
| 550 | Print Operators |
| 551 | Backup Operators |
| 552 | Replicator |

> [!TIP]
> **Local vs domain RID**
> RIDs 500–501 and 544–552 appear on **every** machine and describe local/built-in principals. RIDs from 512 upward in the `512–520` band (Domain Admins, Enterprise Admins, KRBTGT, etc.) only exist under a **domain** SID on a Domain Controller — see [Active-Directory-Domain-Services](../Active-Directory-Domain-Services-AD-DS/Active-Directory-Domain-Services.md) and [Kerberos-Authentication](../Active-Directory-Domain-Services-AD-DS/Kerberos-Authentication.md).

## Identifying the Account

Given an account name and SID, you can determine its type and scope:

| Field | Value |
|-------|-------|
| **Account Name** | `SRV01\Administrator` |
| **SID** | `S-1-5-21-3574371106-2527126871-2867751051-500` |

- The host has the built-in **Administrator** account.
- The account carries the well-known **RID 500**.
- The prefix `SRV01\` (the computer name) shows the account is **local** to the machine.

A local account is displayed as `SRV01\Administrator`, whereas a domain account joined to Active Directory appears as `CONTOSO\Administrator` — the prefix is the NetBIOS domain name rather than the hostname.

## Useful Commands

Show the current logged-in user:

```cmd
whoami
```

Show the current user's SID and account name:

```cmd
whoami /user
```

Check whether the computer is domain joined (returns `True` or `False`):

```powershell
(Get-CimInstance Win32_ComputerSystem).PartOfDomain
```

Display domain or workgroup membership:

```cmd
systeminfo | findstr /i "Domain"
```

List all local users with their SIDs and enabled state:

```powershell
Get-LocalUser | Select-Object Name, SID, Enabled
```

Example output:

```text
Name           SID                                           Enabled
----           ---                                           -------
Administrator  S-1-5-21-3574371106-2527126871-2867751051-500 True
Guest          S-1-5-21-3574371106-2527126871-2867751051-501 False
```

List members of the local Administrators group:

```powershell
Get-LocalGroupMember -Group "Administrators"
```

List all local user accounts and inspect the Administrator account (classic tools):

```cmd
net user
net user administrator
```

## Security Considerations

> [!WARNING]
> **RID 500 is a standing target**
> The built-in RID `500` Administrator is a permanent, non-deletable local admin present on every Windows host, which makes it a favourite target for offensive operators. By default this account is **exempt from UAC remote token filtering** — an attacker who obtains its NT hash or password can authenticate remotely (for example via SMB/`PsExec`) and receive a **full high-integrity token**, enabling [pass-the-hash](../Active-Directory-Domain-Services-AD-DS/NTLM.md) lateral movement. Local accounts that share the same name **and password** across many machines let a single compromised hash unlock the entire fleet.

- Renaming the account provides only marginal cover — the SID still ends in `-500`, so tooling can find it by RID regardless of the name.
- Because credential material for local accounts lives in the SAM hive, a SAM + SYSTEM hive dump exposes the Administrator NT hash for offline cracking or pass-the-hash. See [SAM-vs-NTDS.dit](../Active-Directory-Domain-Services-AD-DS/SAM-vs-NTDS.dit.md).
- Enumerating SIDs (via `whoami /user`, `Get-LocalUser`, or SAM-remote/LSA lookups) is a routine post-exploitation step to map privileged accounts and group membership.

## Best Practices

- Deploy **LAPS** (Local Administrator Password Solution) so every host's built-in Administrator has a unique, rotated password — this defeats password-reuse lateral movement.
- Disable or heavily restrict the built-in RID `500` account and use named, individually accountable admin accounts for daily work.
- Set `LocalAccountTokenFilterPolicy` / enable UAC remote restrictions where feasible so local admins other than RID 500 cannot wield full tokens over the network.
- Rename the built-in Administrator as a minor speed bump, but never rely on renaming alone for protection.
- Audit local Administrators group membership and logons of RID `500` centrally (see [Windows-Audit-Policy](Windows-Audit-Policy.md) and [Windows-Event-Logs](Windows-Event-Logs.md)).

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| `Get-LocalUser` / `Get-LocalGroupMember` not recognized | The `Microsoft.PowerShell.LocalAccounts` module needs PowerShell 5.1+ (Windows 10 / Server 2016+); on older hosts use `net user` and `net localgroup`. |
| Account name shows `HOST\Administrator`, not `DOMAIN\` | The principal is local, not a domain account — confirm domain membership with `(Get-CimInstance Win32_ComputerSystem).PartOfDomain`. |
| Renamed the Administrator but tools still flag RID 500 | Expected — the SID/RID is immutable; identification is by RID, not name. |
| Remote admin logon with a local account is denied | UAC remote token filtering is blocking non-RID-500 local admins; this is by design unless `LocalAccountTokenFilterPolicy` is set. |

## References

- [Security identifiers (SIDs) — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers)
- [Well-known SIDs — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers#well-known-sids)
- [Local Accounts (built-in Administrator) — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/local-accounts)
- [Local Administrator Password Solution (LAPS) — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub
- [User-Management](User-Management.md) — managing local users and groups (GUI)
- [PowerShell-User-Group-Management](PowerShell-User-Group-Management.md) — user/group management with PowerShell
- [Add-User-to-Administrators](Add-User-to-Administrators.md) — granting local admin rights
- [Enable-Guest-Login](Enable-Guest-Login.md) — the built-in Guest account (RID 501)
- [Windows-Audit-Policy](Windows-Audit-Policy.md) — auditing account and logon activity
- [Windows-Event-Logs](Windows-Event-Logs.md) — reading the logs that record admin logons
- [SAM-vs-NTDS.dit](../Active-Directory-Domain-Services-AD-DS/SAM-vs-NTDS.dit.md) — where local vs domain credentials are stored
- [NTLM](../Active-Directory-Domain-Services-AD-DS/NTLM.md) — pass-the-hash and the NT hash behind local auth
- [Active-Directory-Domain-Services](../Active-Directory-Domain-Services-AD-DS/Active-Directory-Domain-Services.md) — domain SIDs and well-known domain RIDs
