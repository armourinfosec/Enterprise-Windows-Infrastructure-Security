# CPU Architecture

**CPU architecture** refers to the design and structure of a computer's **central processing unit (CPU)** — the instruction set it understands, how wide its internal data path is, and how it addresses memory and communicates with the rest of the machine. Think of it as the blueprint that decides what software can run on a given processor and how much memory that software can reach.

## Overview

Every operating system and application is compiled for a specific CPU architecture, so the architecture is the lowest layer that determines software compatibility. It defines the **instruction set** (the vocabulary of low-level commands), the **register/word size** (32-bit vs 64-bit), and the **address bus width** (how much memory the CPU can address directly). These choices ripple all the way up: a 64-bit OS needs a 64-bit CPU, and a payload built for one architecture will not execute on another.

Architecture sits directly above the hardware and firmware described in [Fundamental-Of-Computers](Fundamental-Of-Computers.md) and [Laptop-or-PC-Specifications](Laptop-or-PC-Specifications.md), and directly beneath the [Operating-System](Operating-System.md) that abstracts it for programs. Understanding it is the foundation for reading system specs, choosing the right OS edition, and — in offensive work — building payloads that match the target.

> [!NOTE]
> **The one number that matters most**
> When people say a system is "32-bit" or "64-bit", they are describing the CPU's register/word size. That single property caps the directly addressable memory (~4 GB for 32-bit) and dictates which OS and applications will run.

## Key Elements of CPU Architecture

| Element | Description |
| --- | --- |
| **Instruction Set (ISA)** | The set of low-level commands a CPU understands (e.g., x86, ARM, RISC-V) |
| **Register Size** | Internal storage width (e.g., 32-bit or 64-bit) affecting how much data is handled per operation |
| **Word Size** | How much data the CPU processes at once — typically the same as register size |
| **Address Bus Width** | Determines how much memory the CPU can address directly |
| **Pipeline & Cores** | Affect how efficiently and how many instructions the CPU executes in parallel |

## How It Works — The Instruction Cycle

At its core, a CPU repeats a simple loop for every instruction: **fetch** it from memory, **decode** it into a micro-operation, **execute** it in the arithmetic/logic unit, and **write back** the result. Registers hold the operands and the address of the next instruction; the address bus width sets the ceiling on which memory addresses are reachable during the fetch step.

```mermaid
flowchart LR
    A[Fetch<br/>read instruction<br/>from memory] --> B[Decode<br/>interpret opcode<br/>and operands]
    B --> C[Execute<br/>ALU performs<br/>the operation]
    C --> D[Write Back<br/>store result to<br/>register/memory]
    D --> A
```

The wider the registers and address bus, the more data each cycle can move and the more memory the CPU can reach — which is the practical difference between 32-bit and 64-bit processing.

## Common CPU Architectures

| Architecture | Bit Size | Used By | Notes |
| --- | --- | --- | --- |
| **x86** | 32-bit | Older PCs, embedded devices | Legacy architecture; limited to ~4 GB RAM |
| **x86-64 / AMD64** | 64-bit | Modern PCs, laptops, servers | Backward-compatible with x86; supports large memory |
| **ARM (32/64-bit)** | 32 or 64-bit | Phones, tablets, Apple Silicon Macs | Power-efficient; dominates mobile; Apple M-series chips use ARM64 |
| **RISC-V** | Variable | IoT, research, growing use | Open-source instruction set architecture |
| **PowerPC** | 32/64-bit | Older Macs, IBM systems | Less common today in consumer hardware |

> [!TIP]
> **Why this matters in practice**
> If a CPU is **x86 (32-bit)** it cannot run a 64-bit operating system — the processor simply does not understand those instructions. If a CPU is **ARM-based** (a phone or an Apple M-series Mac), it cannot run traditional x86 software without an emulator/translation layer.

## x86 vs x64 (32-bit vs 64-bit)

### x86 (32-bit)

- Refers to **32-bit architecture**, originating from the Intel 8086 and its successors.
- Can address **up to ~4 GB of RAM**.
- Common in older systems and embedded devices.
- 32-bit operating systems and applications are built for x86 CPUs.

### x64 (64-bit)

- Refers to **64-bit architecture** (also called **x86-64** or **AMD64**).
- Addresses far more memory — the architecture allows a vast address space (well beyond 4 GB; current implementations expose terabytes).
- Uses **larger registers** and processes more data per operation.
- Required for modern applications, games, and high-performance/virtualized workloads.
- Can run both 64-bit and 32-bit software, depending on the OS (on Windows, via the WOW64 subsystem).

### Key Differences

| Feature | x86 (32-bit) | x64 (64-bit) |
| --- | --- | --- |
| Addressable RAM | Max ~4 GB | Far beyond 4 GB (implementation-dependent) |
| Performance | Lower | Higher (better multitasking) |
| Compatibility | 32-bit apps only | 32-bit and 64-bit apps |
| Use case | Older hardware/tasks | Modern PCs, servers, gaming |

## Checking System Architecture

**Windows (GUI):** `Settings > System > About` → **System type**, e.g. `64-bit operating system, x64-based processor`.

**Windows (cmd):** read the processor architecture environment variable.

```cmd
echo %PROCESSOR_ARCHITECTURE%
```

**Windows (PowerShell):** query the OS and current process bitness.

```powershell
[Environment]::Is64BitOperatingSystem
[Environment]::Is64BitProcess
```

**Linux/macOS:** print the machine hardware name.

```bash
uname -m
```

- Output `x86_64` (or `aarch64`/`arm64`) → 64-bit
- Output `i686` or `i386` → 32-bit

## Security Considerations

> [!WARNING]
> **Architecture mismatch breaks payloads — and defenses depend on it**
> Offensive tooling is architecture-specific. A 32-bit (x86) shellcode or implant will not execute inside a 64-bit (x64) process, and vice versa. When generating payloads you must match the target's architecture, and when injecting into or migrating between processes on 64-bit Windows you must account for the **WOW64** boundary between 32-bit and 64-bit processes (separate `System32` vs `SysWOW64` directories and registry redirection). Guessing wrong is a common cause of a payload that "runs" but silently fails.

- **Payload selection** — enumerate the target architecture *before* building an implant; a mismatched architecture is the difference between a session and a crash.
- **WOW64 pitfalls** — a 32-bit process on 64-bit Windows sees redirected file-system and registry paths; commands run from it may not see the true 64-bit system state.
- **Exploit mitigations scale with the architecture** — 64-bit's larger address space makes **ASLR** far more effective (many more bits of entropy), and modern CPUs enforce **DEP/NX**, and kernel protections such as SMEP/SMAP, raising the bar for memory-corruption exploits.
- **Microarchitectural attacks** — speculative-execution flaws (e.g., Spectre/Meltdown-class issues) are properties of the CPU design itself, not the OS, and are mitigated by microcode plus OS/firmware updates.

## Best Practices

- Match the OS and application bitness to the CPU: run a 64-bit OS on 64-bit hardware to use available RAM and stronger mitigations.
- Prefer 64-bit builds of tools and payloads unless the target is confirmed 32-bit.
- Record host and VM CPU architecture in lab notes so exploits and payloads are reproducible.
- Keep CPU microcode and firmware updated to close microarchitectural vulnerabilities.
- On 64-bit Windows, be deliberate about whether a process is 32-bit or 64-bit to avoid WOW64 redirection surprises.

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| "This app can't run on your PC" / not a valid Win32 application | Architecture mismatch — running a 64-bit binary on a 32-bit OS (or vice versa); obtain the matching build |
| 64-bit OS install refused | CPU is 32-bit only — verify with `PROCESSOR_ARCHITECTURE` / `uname -m` |
| Payload runs but no session returns | Payload architecture does not match the target process; regenerate for the correct arch |
| System reports less RAM than installed | 32-bit OS capping usable memory at ~4 GB — install a 64-bit OS |

## References

- [Microsoft Learn — WOW64 (running 32-bit apps on 64-bit Windows)](https://learn.microsoft.com/en-us/windows/win32/winprog64/running-32-bit-applications)
- [Microsoft Learn — Processor architecture](https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/64-bit-and-32-bit-file-systems)
- [Intel 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
- [Arm Architecture Reference Manual](https://developer.arm.com/documentation/ddi0487/latest/)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Fundamental-Of-Computers](Fundamental-Of-Computers.md) — related note (what a computer is and how its parts fit together)
- [Laptop-or-PC-Specifications](Laptop-or-PC-Specifications.md) — related note (CPU is a core spec component)
- [Operating-System](Operating-System.md) — related note (the OS built on top of the CPU architecture)
- [BIOS-and-UEFI](BIOS-and-UEFI.md) — related note (firmware that initializes the CPU at boot)
- [Windows-Operating-System-Editions](Windows-Operating-System-Editions.md) — related note (32-bit vs 64-bit editions)
