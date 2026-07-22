# CACLS Command

`CACLS` (Change Access Control Lists) is a legacy Windows command-line utility used to display or modify the Access Control Lists (ACLs) of files and folders on NTFS volumes. It has been deprecated by Microsoft in favour of the more capable `ICACLS` command.

> [!WARNING]
> **Deprecated command**
> `CACLS` is deprecated on modern Windows. It does not understand many NTFS security features (integrity levels, SDDL, ACL backup/restore) and can corrupt or misreport modern security descriptors. Use [icacls](ICACLS-Command.md) for all new administration and scripting.

## Overview

The `CACLS` command allows administrators to:

- View file and folder permissions
- Grant permissions
- Revoke permissions
- Replace existing ACLs
- Edit (append to) existing ACLs

Unlike `ICACLS`, `CACLS` has limited functionality and does **not** support many modern NTFS security features.

## Syntax

```cmd
cacls filename [/T] [/E] [/C] [/G user:perm]
                [/R user [...]]
                [/P user:perm [...]]
                [/D user [...]]
```

## Parameters

| Parameter | Description |
|-----------|-------------|
| `filename` | File or directory to modify. Wildcards are supported. |
| `/T` | Operates on all matching files in the current directory and subdirectories. |
| `/E` | Edits the ACL instead of replacing it. |
| `/C` | Continues on "access denied" errors. |
| `/G user:perm` | Grants permissions to a user. |
| `/R user` | Revokes permissions from a user (used with `/E`). |
| `/P user:perm` | Replaces permissions for a user. |
| `/D user` | Denies access to a user. |

### Permission codes

| Code | Permission |
|------|------------|
| `R` | Read |
| `C` | Change (Read + Write + Execute) |
| `F` | Full Control |

## Examples

### Viewing permissions

Display ACLs for a file:

```cmd
cacls C:\Data\file.txt
```

Display ACLs for a directory:

```cmd
cacls C:\Data
```

Display permissions recursively:

```cmd
cacls C:\Data /T
```

### Grant permissions

Grant Full Control:

```cmd
cacls C:\Data\file.txt /E /G User:F
```

Grant Read permission:

```cmd
cacls C:\Data\file.txt /E /G User:R
```

Grant Change permission:

```cmd
cacls C:\Data\file.txt /E /G User:C
```

Grant permissions recursively:

```cmd
cacls C:\Data /T /E /G User:F
```

### Replace permissions

Replace the user's permissions instead of appending:

```cmd
cacls C:\Data\file.txt /P User:R
```

### Revoke permissions

Remove all explicit permissions for a user:

```cmd
cacls C:\Data\file.txt /E /R User
```

### Deny access

Deny all access:

```cmd
cacls C:\Data\file.txt /E /D User
```

### Recursive operations

Grant permissions to all files and subdirectories:

```cmd
cacls C:\Data /T /E /G User:F
```

Continue despite errors:

```cmd
cacls C:\Data /T /E /C /G User:R
```

### Display help

```cmd
cacls /?
```

### Named examples

View an ACL:

```cmd
cacls C:\Windows\System32\cmd.exe
```

Grant Full Control:

```cmd
cacls C:\Test /E /G Rahul:F
```

Remove a user's permissions:

```cmd
cacls C:\Test /E /R Rahul
```

Replace with Read-only:

```cmd
cacls C:\Test /P Rahul:R
```

Deny access:

```cmd
cacls C:\Test /E /D Rahul
```

## Enterprise Usage

> [!NOTE]
> **Required privileges**
> Viewing an ACL needs only read access to the object; modifying, granting, denying, or performing recursive changes requires elevation (Administrator) or ownership plus the appropriate NTFS rights.

| Operation | Administrator required |
|-----------|------------------------|
| View permissions | No (if the object is accessible) |
| Modify permissions | Yes (or ownership / appropriate NTFS rights) |
| Grant permissions | Yes |
| Deny permissions | Yes |
| Recursive changes | Usually Yes |

`CACLS` should only be encountered in the enterprise when maintaining **legacy scripts**. It does **not** support:

- Integrity Levels
- Mandatory Integrity Control (MIC)
- Backup and restore of ACLs
- SDDL editing
- Advanced inheritance management
- Modern Windows security descriptors

### Comparison with ICACLS

| Feature | CACLS | ICACLS |
|----------|:-----:|:------:|
| View ACLs | ✅ | ✅ |
| Modify ACLs | ✅ | ✅ |
| Backup ACLs | ❌ | ✅ |
| Restore ACLs | ❌ | ✅ |
| Integrity Levels | ❌ | ✅ |
| SDDL Support | ❌ | ✅ |
| Advanced Inheritance | ❌ | ✅ |
| Modern Windows Support | ❌ | ✅ |

> [!TIP]
> **Migration**
> Replace `cacls` with `icacls` in any script you still maintain. The permission model is richer and the tool is actively supported.

## Security Considerations

> [!WARNING]
> **Incorrect ACL changes are dangerous**
> Bad ACL modifications can result in:
>
> - Unauthorized access to sensitive data
> - Loss of access to files (including by administrators)
> - Application failures
> - Privilege-escalation opportunities

- Prefer `ICACLS` for all new scripts.
- Run Command Prompt **as Administrator** when modifying ACLs.
- Back up permissions before making large changes.
- Use inheritance rather than manually assigning permissions where possible.
- Always validate permissions after making changes.

## Troubleshooting

### The data is invalid

```text
The data is invalid.
```

**Possible causes:**

- Using an incorrect account name.
- Invalid permission syntax.
- The target uses security features unsupported by `CACLS`.
- The ACL is incompatible with the legacy command.

For example:

```cmd
cacls C:\Data /E /G SRV01\rahul:F
```

may fail with:

```text
The data is invalid.
```

Using `ICACLS` instead typically succeeds:

```cmd
icacls C:\Data /grant SRV01\rahul:F
```

### Access is denied

```text
Access is denied.
```

**Possible causes:**

- Not running with elevated privileges.
- The user does not own the object.
- Insufficient NTFS permissions.

## References

- [Microsoft — cacls command reference](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/cacls)
- [Microsoft — icacls (recommended replacement)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [ICACLS-Command](ICACLS-Command.md) — modern replacement for cacls
- [TAKEOWN-Command](TAKEOWN-Command.md) — related note for seizing ownership before editing ACLs
- [NTFS-(New-Technology-File-System)-Permissions](NTFS-(New-Technology-File-System)-Permissions.md) — the permission model cacls edits
- [NTFS-Default-Permissions](NTFS-Default-Permissions.md) — related note
- [NTFS-Permissions-Setup-with-PowerShell](NTFS-Permissions-Setup-with-PowerShell.md) — the PowerShell approach to the same task
