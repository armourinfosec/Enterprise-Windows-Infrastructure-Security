# Non Administrator User Write Permission Locations in Windows

## Overview

**Standard (non-administrator) users** in Windows have write access primarily to their own user profile and certain shared directories. Understanding these writable locations is important for system administration, troubleshooting, software deployment, and security assessments.

> [!IMPORTANT]
> Writable directories are frequently used by legitimate applications for storing user data and temporary files. They may also be abused by attackers for persistence, DLL hijacking, or privilege escalation if applications insecurely load files from these locations.

## Concepts

Standard users can reliably write to their profile tree and the shared `Public` folders. The table below lists the common writable locations.

| Location | Description | Writable by Standard Users |
| --- | --- | --- |
| `%USERPROFILE%` | User profile root directory | Yes |
| `%USERPROFILE%\Desktop` | User's Desktop | Yes |
| `%USERPROFILE%\Documents` | User's Documents | Yes |
| `%USERPROFILE%\Downloads` | User's Downloads | Yes |
| `%USERPROFILE%\Pictures` | User's Pictures | Yes |
| `%USERPROFILE%\Music` | User's Music | Yes |
| `%USERPROFILE%\Videos` | User's Videos | Yes |
| `%USERPROFILE%\AppData\Local` | User-specific application data | Yes |
| `%USERPROFILE%\AppData\Roaming` | Roaming application data | Yes |
| `%LOCALAPPDATA%\Temp` | User temporary directory | Yes |
| `%TEMP%` | User temporary directory (environment variable) | Yes |
| `%TMP%` | User temporary directory (environment variable) | Yes |
| `C:\Users\Public\Desktop` | Shared Desktop | Yes (default permissions) |
| `C:\Users\Public\Documents` | Shared Documents | Yes |
| `C:\Users\Public\Downloads` | Shared Downloads (if present) | Yes |
| `C:\Users\Public\Pictures` | Shared Pictures | Yes |
| `C:\Users\Public\Music` | Shared Music | Yes |
| `C:\Users\Public\Videos` | Shared Videos | Yes |

## Commands

### Environment Variables

Display the current user's writable directories.

Command Prompt:

```cmd
echo %USERPROFILE%
echo %TEMP%
echo %TMP%
echo %LOCALAPPDATA%
echo %APPDATA%
```

PowerShell:

```powershell
$env:USERPROFILE
$env:TEMP
$env:TMP
$env:LOCALAPPDATA
$env:APPDATA
```

### Verify Directory Permissions

PowerShell — display the Access Control List (ACL):

```powershell
Get-Acl "$env:TEMP"
```

Display permissions in a readable format:

```powershell
(Get-Acl "$env:TEMP").Access
```

Command Prompt — view NTFS permissions:

```cmd
icacls "%TEMP%"
```

Example:

```cmd
icacls "C:\Users\Public\Documents"
```

### Check Effective Permissions

Determine whether the current user can create files.

PowerShell:

```powershell
Test-Path "$env:TEMP"
```

Create a test file:

```powershell
New-Item "$env:TEMP\test.txt" -ItemType File
```

Remove the test file:

```powershell
Remove-Item "$env:TEMP\test.txt"
```

### Related Commands

- `icacls`
- `takeown`
- `whoami`
- `whoami /priv`
- `whoami /groups`
- `Get-Acl`
- `Set-Acl`

## Best Practices

- Store user data only within the user's profile.
- Avoid granting write access to system directories.
- Regularly audit NTFS permissions.
- Apply the principle of least privilege.
- Restrict write permissions on application installation directories.
- Validate writable paths during security assessments.

## Security Considerations

Writable directories are commonly examined during security assessments because they may be abused in scenarios such as:

- DLL search-order hijacking
- Application search-path hijacking
- Insecure file permissions
- Scheduled task abuse
- Service misconfigurations
- Startup folder persistence
- Arbitrary file write vulnerabilities
- User-level persistence mechanisms

> [!WARNING]
> A writable directory alone does **not** indicate a security vulnerability. Exploitation typically requires another weakness, such as an application loading executable content from that location without proper validation.

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| `Access is denied` writing to a Public folder | Non-default ACLs applied by hardening | Review with `icacls` and adjust or use a profile path |
| `%TEMP%` not writable | Corrupt profile or quota/disk limits | Verify profile integrity and free space; recreate the Temp folder |
| ACL output hard to read | `Get-Acl` returns full SDDL | Expand with `(Get-Acl path).Access` for per-ACE detail |

## References

- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls>
- <https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows-Basic-Commands](Windows-Basic-Commands.md) — `icacls`, `takeown`, and file management commands
- [Windows-Registry](Windows-Registry.md) — registry autorun keys used for user-level persistence
- [ATTRIB-Command](ATTRIB-Command.md) — file attributes on writable paths
- [8.3-Filename-(Short-File-Name)](8.3-Filename-(Short-File-Name).md) — short filenames and path discovery
