# ATTRIB Command

`attrib` is a built-in Windows console command that displays, sets, or clears the file-system attributes of files and directories — Read-only, Archive, System, Hidden, Not-content-indexed, and (with `/L`) the attributes of a symbolic link itself rather than its target.

## Overview

Every file and directory on an NTFS or FAT volume carries a set of attribute flags stored in the file record. `attrib` is the classic CMD tool for reading and toggling those flags without opening the file's **Properties** dialog. It operates on file *metadata* only — it does **not** touch NTFS discretionary access control lists (DACLs); use [icacls](Windows-Basic-Commands.md) for permissions.

Running `attrib` with no arguments in the current directory lists every file and its currently-set attributes:

```cmd
attrib
```

```text
A          C:\Users\admin\notes.txt
A    SH    C:\Users\admin\desktop.ini
    R      C:\Users\admin\readonly.cfg
```

Each letter in the left-hand column indicates a set attribute (Archive, Read-only, System, Hidden, etc.).

## Syntax

```cmd
attrib [+attribute | -attribute] [drive:][path][filename] [/S [/D] [/L]]
```

- `+attribute` sets a flag, `-attribute` clears it.
- `[drive:][path][filename]` selects the target; wildcards (`*`, `?`) are allowed.
- Switches (`/S`, `/D`, `/L`) control recursion and link handling.

Display the built-in help:

```cmd
attrib /?
```

## Parameters

| Parameter | Meaning |
| --- | --- |
| `+R` / `-R` | Set / clear the **Read-only** attribute |
| `+A` / `-A` | Set / clear the **Archive** attribute (marks a file as changed since last backup) |
| `+S` / `-S` | Set / clear the **System** attribute |
| `+H` / `-H` | Set / clear the **Hidden** attribute |
| `+I` / `-I` | Set / clear the **Not content indexed** attribute (excludes the file from the Windows Search index) |
| `[drive:][path][filename]` | File(s) or directory to process; supports `*` and `?` wildcards |
| `/S` | Process matching files in the current folder **and all subdirectories** |
| `/D` | Process folders as well (used together with `/S`) |
| `/L` | Operate on the attributes of the **symbolic link itself**, not the file the link points to |

> [!NOTE]
> The `+O` (Offline), `+P` (Pinned), `+U` (Unpinned), `+B` (SMR Blob), `+V` (Integrity) and `+X` (No scrub) attributes exist on newer Windows/ReFS builds. They are situational and version-dependent, so verify availability on your target before relying on them. `# untested`

## Examples

Show the attributes of a single file:

```cmd
attrib file.txt
```

Hide a file:

```cmd
attrib +H file.txt
```

Remove the hidden attribute:

```cmd
attrib -H file.txt
```

Make a file read-only, then remove read-only:

```cmd
attrib +R config.ini
attrib -R config.ini
```

Set both Hidden and System (the combination Windows uses for protected OS files):

```cmd
attrib +H +S secret.dat
```

Recursively hide every `.log` file under a tree, including inside subfolders:

```cmd
attrib +H C:\Data\*.log /S
```

Apply Hidden + System recursively to files **and** directories:

```cmd
attrib +H +S C:\Data\* /S /D
```

Clear Read-only, Hidden, and System from a stubborn file so it can be deleted or edited:

```cmd
attrib -R -H -S "C:\Users\admin\stuck.dat"
```

Reveal everything hidden in a directory tree (useful triage during incident response):

```cmd
attrib -H -S C:\Suspect\* /S /D
```

## Enterprise Usage

- **Backup workflows** — the Archive (`A`) bit is the traditional signal to incremental backup tools. Clearing it after a backup (`attrib -A`) and letting Windows re-set it on the next write is the classic incremental-backup mechanism; most modern tools track change journals instead but still honor the flag.
- **Deployment scripts** — logon and provisioning `.bat` files frequently mark configuration or template files Read-only (`+R`) to prevent accidental user modification, or Hidden (`+H`) to declutter mapped-drive roots.
- **Search-index tuning** — `+I` on large log or archive folders keeps them out of the Windows Search index, reducing indexer load on file servers.
- **Recovering locked files** — administrators use `attrib -R -H -S` to clear the protected-file combination before repairing or removing OS-planted files (e.g. a corrupt `desktop.ini`).

> [!TIP]
> `attrib` pairs naturally with `dir` — use `dir /A:H` to list only hidden items, or `dir /A:S` for system items, to confirm the result of an `attrib` change.

## Security Considerations

> [!WARNING]
> Setting **Hidden + System** (`attrib +H +S`) on a file removes it from a default `dir` listing *and* from Explorer even when "Show hidden files" is enabled (Explorer additionally requires unchecking "Hide protected operating system files"). Malware and post-exploitation tooling routinely use this combination to conceal dropped payloads, staging directories, and exfiltration archives.

- Attribute-based hiding is **security by obscurity** — it changes visibility, not access. It provides no confidentiality and is trivially reversed with `attrib -H -S` or `dir /A`.
- During triage, always enumerate hidden/system content: `dir /A:HS /S /B` from a suspect root, or `attrib /S /D` to dump attributes across a tree.
- `attrib` **cannot** change NTFS permissions/ownership. To lock down or open up who may read/write a file, use `icacls` (permissions) and `takeown` (ownership) — see [Non-Administrator-User-Write-Permission-Locations-in-Windows](Non-Administrator-User-Write-Permission-Locations-in-Windows.md).
- The Read-only bit does **not** protect a file from deletion by a user who has NTFS Modify/Delete rights on the parent directory; it only blocks in-place overwrite through APIs that honor it.

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| `Access denied - <file>` | File is in use, or the account lacks NTFS write rights | Close the process holding the file; run the prompt elevated; verify DACL with `icacls` |
| `+H`/`+S` file still shows in Explorer | "Show hidden files" and/or "Hide protected OS files" toggles are set to reveal | Expected — attributes hide from *default* views only |
| Attribute change ignored on a whole folder | `/S` used without `/D`, so directories were skipped | Add `/D` to include directories: `attrib +H C:\Data\* /S /D` |
| Cannot clear Read-only on many files at once | Wildcard didn't recurse | Add `/S` (and `/D` for folders): `attrib -R C:\Data\* /S /D` |
| Change hit the link target instead of the symlink | `/L` omitted | Use `/L` to act on the symbolic link's own attributes |

## References

- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/attrib>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dir>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows-Basic-Commands](Windows-Basic-Commands.md) — core CMD file-management commands including `dir`, `icacls`, and `takeown` — related note
- [8.3-Filename-(Short-File-Name)](8.3-Filename-(Short-File-Name).md) — short-name resolution that interacts with hidden/system paths — related note
- [Non-Administrator-User-Write-Permission-Locations-in-Windows](Non-Administrator-User-Write-Permission-Locations-in-Windows.md) — NTFS permissions and writable locations that `attrib` cannot alter — related note
