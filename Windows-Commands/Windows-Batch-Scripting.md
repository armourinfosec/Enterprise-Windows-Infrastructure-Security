# Windows Batch Scripting

Windows batch scripting is the native automation language of the Command Prompt (`cmd.exe`): plain-text `.bat` or `.cmd` files containing a sequence of console commands, plus a small control-flow layer (variables, conditionals, loops, labels, and subroutine calls) interpreted line-by-line by the command processor.

## Overview

A batch file is any text file whose lines are commands you could type at a `cmd.exe` prompt. Double-clicking it, or running its name, feeds those lines to the interpreter in order. Batch remains ubiquitous for logon scripts, scheduled-task wrappers, build steps, and quick administrative glue because it needs no runtime beyond `cmd.exe`, which ships on every Windows install.

```cmd
@echo off
echo Hello, %USERNAME%!
echo Today is %DATE% at %TIME%
pause
```

- `.bat` — the legacy extension (DOS heritage).
- `.cmd` — the Windows NT extension; behaves identically for most purposes and is preferred on modern systems (subtle differences exist in how `ERRORLEVEL` is set by some internal commands).

> [!TIP]
> Run a batch file from an existing console (`myscript.bat`) rather than double-clicking, so the window stays open and you can read errors. Use `pause` at the end only for interactive scripts.

## Concepts

- **Echoing** — by default the interpreter prints each command before running it. `@echo off` at the top silences this; the leading `@` hides the `echo off` line itself.
- **Comments** — `REM` is the portable comment command; `::` (a broken label) is a common shorthand but must not be used inside a `for`/`if` block or it can break parsing.
- **Variables** — created with `set`, expanded by wrapping the name in percent signs: `%NAME%`.
- **Arguments** — values passed on the command line are available as `%1` through `%9`, with `%*` for all of them and `%0` for the script's own name.
- **Exit codes** — commands report success/failure through the `ERRORLEVEL` pseudo-variable, which drives conditional logic.
- **Scope** — `setlocal` / `endlocal` bound where variable changes are visible, so a script doesn't pollute the parent environment.

## Syntax

### Comments

```cmd
REM This is a standard comment
:: This is a shorthand comment (avoid inside for/if blocks)
```

### Variables

```cmd
set NAME=Armour
echo %NAME%

REM Prompt the user for input
set /p TARGET=Enter target IP:
echo You entered %TARGET%

REM Arithmetic requires /a
set /a TOTAL=5+7
echo %TOTAL%
```

`setlocal`/`endlocal` and delayed expansion (needed when a variable changes *inside* a loop and must be read in the same iteration):

```cmd
@echo off
setlocal enabledelayedexpansion
set COUNT=0
for %%F in (*.txt) do (
    set /a COUNT+=1
    echo File !COUNT!: %%F
)
echo Total: %COUNT%
endlocal
```

> [!NOTE]
> Inside a parenthesised block, ordinary `%VAR%` is expanded **once**, before the block runs. To read a value that changes each iteration you must enable `setlocal enabledelayedexpansion` and reference it as `!VAR!`.

### Parameters

```cmd
@echo off
echo Script name : %0
echo First arg   : %1
echo Second arg  : %2
echo All args    : %*
```

### Conditionals (if / else)

```cmd
@echo off
if "%1"=="" (
    echo No argument supplied.
    goto :eof
)

if exist "C:\Data\report.txt" (
    echo Report found.
) else (
    echo Report missing.
)

REM Compare numbers
if %ERRORLEVEL% NEQ 0 echo Previous command failed.
```

Comparison operators: `EQU` `NEQ` `LSS` `LEQ` `GTR` `GEQ` (numeric), plus `==` for string equality and `/I` for case-insensitive string compares.

### Loops (for)

```cmd
REM Iterate over a set / wildcard
for %%F in (*.log) do echo Processing %%F

REM /L numeric counter: (start,step,end)
for /L %%N in (1,1,5) do echo Count %%N

REM /R recurse a directory tree
for /R C:\Data %%F in (*.txt) do echo Found %%F

REM /F parse file contents or command output, line by line
for /F "tokens=1,2 delims=," %%A in (data.csv) do echo Col1=%%A Col2=%%B
```

> [!IMPORTANT]
> On the command line a `for` variable uses a single percent (`%F`); **inside a batch file it must be doubled** (`%%F`). This is one of the most common batch bugs.

### Labels, goto, and call

```cmd
@echo off
goto main

:greet
echo Hello from a subroutine
goto :eof

:main
call :greet
echo Back in main
```

- `goto label` jumps to `:label`.
- `goto :eof` exits the current script (or subroutine) — `:eof` is a built-in end-of-file target.
- `call :label` runs a labelled block as a subroutine and returns; `call other.bat` runs another batch file and returns (plain `other.bat` would transfer control and not come back).

### Redirection and pipes

```cmd
REM Overwrite / append to a file
echo Log start > run.log
echo more data >> run.log

REM Discard output / suppress errors
ping 127.0.0.1 >nul 2>&1

REM Pipe output into another command
tasklist | find /i "explorer.exe"
```

`>` overwrites, `>>` appends, `2>&1` merges stderr into stdout, `nul` discards, and `|` pipes one command's output into another.

## Parameters / Built-in Commands

| Command / token | Purpose |
| --- | --- |
| `@echo off` | Stop the interpreter echoing each command line |
| `REM` / `::` | Comment (rest of line ignored) |
| `set` | Define or display environment variables (`/p` prompt, `/a` arithmetic) |
| `%1`–`%9`, `%*`, `%0` | Positional arguments, all args, script name |
| `setlocal` / `endlocal` | Begin / end a local scope for environment changes |
| `if` / `else` | Conditional execution (`exist`, `defined`, `EQU/NEQ/...`, `/I`) |
| `for` | Loop over sets, files (`/F`), numbers (`/L`), or trees (`/R`) |
| `goto` / `:label` | Unconditional jump to a label; `goto :eof` exits |
| `call` | Invoke a subroutine (`call :label`) or another script and return |
| `%ERRORLEVEL%` | Exit code of the last command (0 = success by convention) |
| `pause` | Wait for a keypress ("Press any key to continue...") |
| `exit /b n` | Exit the current script returning code `n` |
| `shift` | Shift positional arguments left (`%2`→`%1`, ...) |

## Examples

### Timestamped backup with a loop

```cmd
@echo off
setlocal enabledelayedexpansion
REM Copy every .docx under a source tree into a dated backup folder.

set SRC=C:\Users\%USERNAME%\Documents
set DEST=D:\Backup\%DATE:/=-%

if not exist "%DEST%" mkdir "%DEST%"

for /R "%SRC%" %%F in (*.docx) do (
    echo Backing up %%~nxF
    copy "%%F" "%DEST%" >nul
)

echo Backup complete. Files stored in %DEST%
endlocal
```

`%%~nxF` expands to just the file **n**ame + e**x**tension of the loop variable — one of several modifiers (`~n` name, `~x` extension, `~dp` drive+path, `~f` full path, `~z` size).

### Host ping sweep

```cmd
@echo off
setlocal
REM Ping the first 20 hosts on a /24 and report which reply.

set SUBNET=192.168.1

for /L %%I in (1,1,20) do (
    ping -n 1 -w 300 %SUBNET%.%%I >nul
    if !ERRORLEVEL! EQU 0 (
        echo %SUBNET%.%%I is UP
    ) else (
        echo %SUBNET%.%%I is down
    )
)
endlocal
```

> [!WARNING]
> This sweep needs delayed expansion to read `!ERRORLEVEL!` inside the loop — add `setlocal enabledelayedexpansion` (shown combined here as `setlocal` for brevity in the counter example; enable delayed expansion when reading changing values). `# untested`

### Argument-driven wrapper

```cmd
@echo off
if "%1"=="" (
    echo Usage: %0 ^<hostname^>
    exit /b 1
)
echo Resolving %1 ...
nslookup %1
exit /b %ERRORLEVEL%
```

The `^` before `<` and `>` escapes those redirection characters so they print literally.

## Enterprise Usage

- **Logon / startup scripts** — batch files assigned through Group Policy map drives (`net use`), sync time, and set environment for users at logon.
- **Scheduled Task wrappers** — Task Scheduler runs a `.cmd` that sequences several tools, captures output with redirection, and returns an exit code the scheduler can act on.
- **Build and deployment glue** — CI agents and installers still lean on batch for pre/post steps because it needs no interpreter install.
- **Standardized exit codes** — return `exit /b 0` on success and non-zero on failure so orchestration (SCCM, Task Scheduler, CI) can branch correctly.

> [!TIP]
> For anything with real data structures, error handling, or remote management, prefer PowerShell — see [PowerShell-Commands-for-Penetration-Testing](PowerShell-Commands-for-Penetration-Testing.md). Reach for batch when you need a dependency-free wrapper that runs anywhere `cmd.exe` exists.

## Security Considerations

> [!WARNING]
> Batch files are plain text and run with the caller's privileges. A writable script referenced by a scheduled task or logon policy is a classic privilege-escalation and persistence vector — an attacker who can edit it executes code as whoever runs it. Lock down NTFS permissions on script paths.

- **Untrusted input** — arguments and file contents fed into `set`/`for` can inject additional commands. Quote variables (`"%1"`), validate before use, and never `call`/`goto` a label derived from user input.
- **Credential exposure** — passwords hard-coded in a `.bat` (e.g. `net use ... /user:admin P@ss`) are stored in clear text and appear in process listings and command history. Avoid embedding secrets.
- **Persistence abuse** — startup-folder `.bat` files, `Run`-key batch launchers, and logon scripts are common malware footholds; audit them during IR. See [Non-Administrator-User-Write-Permission-Locations-in-Windows](Non-Administrator-User-Write-Permission-Locations-in-Windows.md) for writable paths that enable this.
- **Execution transparency** — unlike PowerShell, classic batch has minimal built-in logging; enable command-line process auditing (Event ID 4688) to capture what batch-spawned processes ran.

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| Loop variable prints literally (`%%F`) or errors | Single `%` used in a file, or double `%%` used at the prompt | Use `%%F` in `.bat` files, `%F` interactively |
| Variable inside a loop always shows its pre-loop value | Standard `%VAR%` expands once before the block | Enable `setlocal enabledelayedexpansion` and use `!VAR!` |
| `::` comment breaks a `for`/`if` block | `::` is a label; labels are invalid inside blocks | Use `REM` for comments inside blocks |
| Script window flashes and closes | Double-clicked, no `pause`, error at end | Run from an open console, or add `pause` while debugging |
| `if %ERRORLEVEL%==0` never true after a pipe | A pipe sets `ERRORLEVEL` from the *last* command in the pipe | Test the specific command directly, or restructure without the pipe |
| Path with spaces splits into multiple args | Unquoted path | Wrap in quotes: `"C:\Program Files\app"` |

## References

- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/cmd>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/for>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/set_1>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/if>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows-Shell](Windows-Shell.md) — the `cmd.exe` interpreter that runs batch files — related note
- [Windows-Basic-Commands](Windows-Basic-Commands.md) — the individual CMD commands scripted together in batch — related note
- [PowerShell-Commands-for-Penetration-Testing](PowerShell-Commands-for-Penetration-Testing.md) — the modern object-oriented alternative to batch — related note
