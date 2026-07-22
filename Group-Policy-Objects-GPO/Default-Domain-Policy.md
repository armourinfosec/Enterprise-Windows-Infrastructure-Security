# Default Domain Policy

The **Default Domain Policy** is a **Group Policy Object (GPO)** that is automatically created when an Active Directory domain is established. It is linked at the **domain root level** and applies to all users and computers in the domain unless overridden by a GPO with higher precedence.

## Overview

- The Default Domain Policy is linked at the domain root:

    ```text
    DC=yourdomain,DC=com
    ```

- This policy applies to **all users and computers** within the domain unless overridden by another GPO with higher precedence.
- It is primarily intended to manage **domain-wide security settings**, especially account-related policies.

> [!IMPORTANT]
> **Scope of use**
> The Default Domain Policy should be reserved for **Password**, **Account Lockout**, and **Kerberos** policies. Put everything else in dedicated GPOs linked to specific OUs.

## Concepts

### Purpose of the Default Domain Policy

The Default Domain Policy is mainly used to:

- Enforce **Password Policies**
- Configure **Account Lockout Policies**
- Define **Kerberos Authentication Policies**
- Establish a consistent **security baseline**
- Apply domain-wide authentication and security settings

## Configuration

### Account Policies

> `Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies`

| Subcategory | Description |
| --- | --- |
| Password Policy | Controls password complexity, expiration, reuse, and length |
| Account Lockout Policy | Locks accounts after repeated failed login attempts |
| Kerberos Policy | Controls Kerberos authentication ticket settings |

> [!NOTE]
> **Screenshot**
> ![Group Policy Management Editor showing Account Policies expanded under Computer Configuration > Windows Settings > Security Settings, with Password Policy, Account Lockout Policy, and Kerberos Policy nodes](placeholder-screenshot)

#### Password Policy

> `Account Policies > Password Policy`

| Setting | Description | Recommended Value |
| --- | --- | --- |
| Enforce password history | Prevents reuse of old passwords | `24 passwords remembered` |
| Maximum password age | Forces periodic password changes | `60–90 days` |
| Minimum password age | Prevents immediate password reuse | `1 day` |
| Minimum password length | Defines minimum password size | `14 characters` |
| Password must meet complexity requirements | Requires uppercase, lowercase, numbers, and symbols | `Enabled` |
| Store passwords using reversible encryption | Allows password recovery (insecure) | `Disabled` |
| Minimum password length audit | Audits compliance with minimum length | `14 characters` |

#### Account Lockout Policy

> `Account Policies > Account Lockout Policy`

| Setting | Description | Recommended Value |
| --- | --- | --- |
| Account lockout threshold | Failed login attempts before lockout | `5 attempts` |
| Account lockout duration | Time account remains locked | `15 minutes` |
| Reset account lockout counter after | Time before failed attempts reset | `15 minutes` |
| Allow Administrator account lockout | Allows built-in admin account lockout | `Disabled` |

#### Kerberos Policy

> `Account Policies > Kerberos Policy`

| Setting | Description | Recommended Value |
| --- | --- | --- |
| Enforce user logon restrictions | Checks permissions before issuing tickets | `Enabled` |
| Maximum lifetime for user ticket (TGT) | Validity of TGT ticket | `10 hours (600 minutes)` |
| Maximum lifetime for service ticket | Service ticket validity | `600 minutes` |
| Maximum lifetime for user ticket renewal | Renewal period for TGTs | `7 days` |
| Maximum tolerance for computer clock synchronization | Allowed clock skew | `5 minutes` |

### Security Options

> `Security Settings > Local Policies > Security Options`

| Setting | Description | Recommended Value |
| --- | --- | --- |
| Accounts: Administrator account status | Enables/disables built-in Administrator account | `Disabled` |
| Accounts: Guest account status | Enables/disables Guest account | `Disabled` |
| Interactive logon: Do not display last user name | Hides username at logon screen | `Enabled` |
| Interactive logon: Message title | Logon banner title | `Set according to policy` |
| Interactive logon: Message text | Logon warning/banner text | `Set according to policy` |

### Windows Firewall Settings

> `Administrative Templates > Network > Network Connections > Windows Firewall`

| Setting | Description | Recommended Value |
| --- | --- | --- |
| Protect all network connections | Enables Windows Firewall | `Enabled` |
| Allow inbound file and printer sharing exception | Allows file/printer sharing | `Disabled` unless required |
| Allow inbound remote administration exception | Allows remote administration | `Disabled` unless required |

### Audit Policy

> `Security Settings > Local Policies > Audit Policy`

| Setting | Description | Recommended Value |
| --- | --- | --- |
| Audit logon events | Tracks user logon activity | `Success and Failure` |
| Audit account logon events | Tracks domain authentication events | `Success and Failure` |
| Audit object access | Monitors file/folder access | `Success and Failure` |
| Audit policy change | Tracks changes to audit settings | `Success and Failure` |

### User Rights Assignment

> `Security Settings > Local Policies > User Rights Assignment`

| Setting | Description | Recommended Value |
| --- | --- | --- |
| Log on locally | Users allowed physical logon | `Administrators, Users` |
| Access this computer from the network | Users allowed network access | `Administrators, Users` |
| Deny logon locally | Blocks local logon | `Guests` |
| Deny logon through Remote Desktop Services | Blocks RDP access | `Guests` |

## Administration

### Audit the Policy

```powershell
Get-GPOReport -Name "Default Domain Policy" -ReportType HTML -Path "C:\Reports\DDP_Report.html"
```

### Back Up the Policy

```powershell
Backup-GPO -Name "Default Domain Policy" -Path "C:\GPOBackup"
```

### Restore the Default Domain Policy

If the policy becomes corrupted or heavily misconfigured:

```powershell
dcgpofix /target:Domain
```

> [!WARNING]
> **Destructive command**
> `dcgpofix` restores the Default Domain Policy to its original default state, discarding all customizations. Back up first and only use it for recovery.

## Security Considerations

### What NOT to Modify in the Default Domain Policy

Avoid configuring unrelated settings in the Default Domain Policy.

| Reason | Explanation |
| --- | --- |
| Centralized impact | Changes affect the entire domain |
| Risk of lockouts | Incorrect settings may block administrators |
| Difficult troubleshooting | Misconfigurations can impact all systems |

> [!IMPORTANT]
> **Use dedicated GPOs instead**
> Create separate GPOs linked to specific OUs for software deployment, login scripts, drive mappings, printer deployment, and desktop restrictions.

### Important Notes

- Only **one Password Policy** and **one Account Lockout Policy** can apply at the domain level through the Default Domain Policy.
- Fine-Grained Password Policies (FGPP) can be used for different password requirements for specific users or groups.
- Avoid disabling or deleting the Default Domain Policy.
- Always test major policy changes in a lab environment before deploying to production.

## Best Practices

- Use the Default Domain Policy primarily for:
    - Password Policies
    - Account Lockout Policies
    - Kerberos Policies
- Create separate GPOs for:
    - Workstation settings
    - Server hardening
    - Application deployment
    - User environment configurations
- Regularly audit and back up the policy.

## References

- Microsoft Learn — Default Domain Policy and Default Domain Controllers Policy: <https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/managing-the-default-domain-policy>
- Microsoft Learn — `dcgpofix`: <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dcgpofix>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Group-Policy(GPO)](Group-Policy(GPO).md) — core Group Policy concepts — related note
- [Domain-Based-Group-Policy-Configuration](Domain-Based-Group-Policy-Configuration.md) — broader GPO configuration in the domain — related note
- [GPO-Processing-Order](GPO-Processing-Order.md) — how domain-level policy is overridden by OU-level GPOs — related note
- [Authentication-Methods-in-Windows](../Web-Server-IIS/Authentication-Methods-in-Windows.md) — the Kerberos and password mechanisms this GPO governs — related note
