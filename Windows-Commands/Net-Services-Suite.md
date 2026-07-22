# Net Services Suite

The **`net`** command is one of the most widely used built-in Windows administration tools, exposing users, groups, shares, sessions, and service control through a single command family usable from any Command Prompt.

## Overview

The **`net`** command suite is a family of built-in Windows utilities for managing local and domain accounts, groups, shares, sessions, services, and network configuration from the command line. Because it ships on every Windows host and requires no additional tooling, it is a core administrative tool and a classic **living-off-the-land** technique — equally useful for reconnaissance and lateral movement during authorized testing. It overlaps with [Service-Controller-Utility-Commands](Service-Controller-Utility-Commands.md) for service control, [WMIC-Commands](WMIC-Commands.md) for WMI queries, and is a staple of [Network-Enumeration](Network-Enumeration.md).

Each function is invoked as a subcommand (`net use`, `net view`, `net user`, …), and every subcommand accepts `/?` for its own syntax help.

## Subcommand Reference

| Subcommand | Purpose |
| --- | --- |
| `net accounts` | View/set password and lockout policy |
| `net user` | List, create, modify, or delete user accounts |
| `net localgroup` | Manage local groups and their membership |
| `net group` | Manage domain global groups (on a DC or with `/domain`) |
| `net computer` | Add/remove a computer from a domain (on a DC) |
| `net use` | Connect to / disconnect from shared resources |
| `net view` | List computers and shared resources on the network |
| `net share` | List or manage local shared resources |
| `net session` | List or disconnect inbound network sessions |
| `net file` | List or close files opened over the network |
| `net start` / `net stop` | Start or stop a Windows service |
| `net config` | Display/modify Workstation or Server service settings |
| `net statistics` | Show Workstation or Server service statistics |
| `net time` | Display or synchronize network time |

> [!TIP]
> **Ask any subcommand for help**
> Every function documents itself. `net /?` lists the family; `net help <subcommand>` (or `net <subcommand> /?`) prints full syntax — for example `net user /?` or `net use /?`.

### General Help

```cmd
net /?
```

## Users, Groups, and Accounts

List and inspect local user accounts:

```cmd
net user
```

```cmd
net user Administrator
```

Query a domain account (from a domain-joined host):

```cmd
net user jsmith /domain
```

List local groups, and enumerate the members of a privileged group:

```cmd
net localgroup
```

```cmd
net localgroup Administrators
```

Enumerate domain groups (run on a Domain Controller or with `/domain`):

```cmd
net group "Domain Admins" /domain
```

Review the account/password and lockout policy:

```cmd
net accounts
```

```cmd
net accounts /domain
```

Add or remove a computer account in the domain (Domain Controller):

```cmd
net computer \\WORKSTATION01 /add
```

## Shares and Remote Connections

Connect using specific user credentials:

```cmd
net use \\192.168.1.61 /user:Administrator @rmour123
```

Connect to a remote share with a drive letter:

```cmd
net use t: \\192.168.1.61\data /user:Administrator @rmour123
```

Disconnect the mapped drive:

```cmd
net use t: /delete
```

Display a list of local shares:

```cmd
net share
```

List the shares on a remote computer:

```cmd
net view \\192.168.1.61
```

List all shares on a remote computer, including hidden administrative shares:

```cmd
net view \\192.168.1.61 /all
```

## Sessions, Files, and Services

View active inbound network sessions (who is connected to this host's shares):

```cmd
net session
```

View files opened over the network:

```cmd
net file
```

Start or stop a service by its service name:

```cmd
net start Spooler
```

```cmd
net stop Spooler
```

## Configuration and Time

View server and workstation configuration:

```cmd
net config server
```

```cmd
net config WORKSTATION
```

Show Server or Workstation service statistics:

```cmd
net statistics server
```

Display or synchronize network time against a host:

```cmd
net time
```

```cmd
net time \\DC01 /set
```

## Security Considerations

The `net` family is dual-use: the same commands that administer a host are the ones an attacker runs first after landing a shell, because they are already present (no tool drop) and blend into normal administrative activity.

> [!WARNING]
> **`net` is a living-off-the-land staple**
> - **Credential exposure** — `net use \\host /user:name password` places the plaintext password on the command line, where it is visible in process listings and, historically, shell history. Prefer prompting (omit the password so `net` asks) over inline credentials.
> - **Reconnaissance** — `net user /domain`, `net group "Domain Admins" /domain`, `net localgroup Administrators`, and `net accounts` are the classic first-move domain and privilege enumeration commands.
> - **Hidden shares** — `net view \\host /all` reveals administrative shares (`C$`, `ADMIN$`, `IPC$`) that are prime targets for lateral movement.
> - **Lateral movement / persistence** — mapping `C$`/`ADMIN$` with `net use`, adding accounts to privileged groups with `net localgroup Administrators <user> /add`, and controlling services with `net start`/`net stop` are common attacker actions.

Detection and defensive notes:

- Baseline and alert on `net use` connections to `C$`/`ADMIN$` and on `net group`/`net localgroup` enumeration bursts — sudden group-membership queries are a strong recon signal.
- Network-share access is recorded by Windows Security **Event ID 5140** (a network share object was accessed); interactive/network logons appear as **Event ID 4624** (logon type 3 for share access). Group-membership changes made via `net localgroup /add` surface as account-management events (e.g. **4732** — a member was added to a security-enabled local group).
- Command-line/process-creation logging (**Event ID 4688** with command line auditing, or Sysmon Event ID 1) captures the actual `net` invocations for hunting.

## Best Practices

- Use `net accounts` to review and enforce password and lockout policy.
- Prefer prompted or credential-manager authentication over inline `/user:password` so secrets never appear in process listings or history.
- Always disconnect mapped drives with `net use <drive>: /delete` when finished, and use `/persistent:no` to avoid reconnecting mappings on reboot.
- Combine `net view` with `net share` to reconcile advertised and actual shares.
- Remember `net` is a common LOLBin — log and baseline `net`, `sc`, `wmic`, and `reg` usage rather than treating them as benign.

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| `System error 5 has occurred` | Access denied / insufficient privileges — supply valid credentials with `/user:` or run from an elevated prompt |
| `System error 53` on `net view` | Host unreachable or file/printer sharing disabled — verify connectivity and that SMB (TCP 445) is open |
| `System error 1219` / multiple-connections error | Conflicting existing session to the same host under different credentials — drop it with `net use \\host /delete` first |
| Mapped drive persists after reboot | Drive mapped as persistent — remap with `/persistent:no` or delete with `net use <drive>: /delete` |
| `The service name is invalid` on `net start` | Wrong service **name** (not display name) — look it up with `sc query` (see [Service-Controller-Utility-Commands](Service-Controller-Utility-Commands.md)) |

## References

- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/net-use>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/net-view>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/net-user>
- <https://lolbas-project.github.io/lolbas/Binaries/Net/>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Network-Enumeration](Network-Enumeration.md) — `net view`/`net share` for host and share discovery
- [Service-Controller-Utility-Commands](Service-Controller-Utility-Commands.md) — `net start`/`net stop` alongside `sc` for service control
- [WMIC-Commands](WMIC-Commands.md) — WMI queries that complement `net` enumeration
- [Windows-Basic-Commands](Windows-Basic-Commands.md) — the wider set of everyday CMD commands
