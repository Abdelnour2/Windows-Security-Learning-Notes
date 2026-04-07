# Introduction
This repository contains a documentation of all my notes while I'm learning Windows System Security!

## Table of Content:
You can find the Table of Content [here](Table-of-Content.md)

## Why I'm starting Over?
It's right that I already started learning and practicing Windows Security and did some projects/cracks, ... But I was thinking about the way I'm doing projects and researches, and noticed that I'm jumping around topics very frequently and randomly, sometimes Ring 3, sometimes Ring 0, sometimes Program Exploits, sometimes researches ... I'm not following a structured approach that helps me explore fully and professionally each area of Windows Security! Also I feel that I'm skipping important knowledge or taking some shortcuts which I don't want.

For that reason, I used Gemini to make me a professional roadmap that reflects my current goals of becoming a professional Red Teamer in Windows System Security Engineering and Research where I work on Reverse Engineering, Anti-Cracking, Malware/Rootkit Analysis and Development, Binary Exploitation!

This is the Roadmap give to me:
### Phase 1: Foundations
- C Programming: Learning C with focus on memory management, pointers, structures, and how the stack and heap actually work.
- x86_64 Assembly: Learning Intel syntax. Understanding registers (RAX, RBX, …), instruction sets, and the calling conventions.
- Computer Architecture: Understanding how the CPU interacts with RAM. Learning about the Fetch-Decode-Execute cycle and the difference between User Mode (Ring 3) and Kernel Mode (Ring 0).

### Phase 2: Windows Internals and User Mode
- PE File Format: Understanding the Portable Executable Format (Headers, Sections, IAT, EAT, …) fully.
- Win32 API: Learning to interact with Windows directly. Focusing on Process, Thread, Memory, and Handle management.
- Virtual Memory: Learning about Page Tables, VAD (Virtual Address Descriptors), and how Windows isolates one process from another.
- Tools: x64dbg for dynamic debugging and PE-Bear for static file analysis.

### Phase 3: Reverse Engineering
- Static Analysis: Master Ghidra or IDA Pro. Learning how to identify code patterns (if/else, loops, switch cases, …) in a decompiler.
- Cracking Logic: Practicing “Crackmes”. Learning how to bypass license checks, find hardcoded keys, and patch binaries to change their behavior.
- Anti-Debugging: Learning how software tries to hide from debuggers (like using IsDebuggerPresent, Timing Checks, …) and how to defeat those protections.

### Phase 4: The Kernel and Driver Development
- Windows Driver Model (WDM/KMDF): Learning to write basic drivers. Understanding the I/O Request Packet (IRP) flow and how drivers communicate with User-Mode apps.
- Kernel Debugging: Learning how to set-up dual machines (test machine, and main machine) to deploy the driver on the test machine and test it using a debugger (WinDbg) on the main machine.
- Hooking Theory: Learning how to intercept system calls (SSDT Hooking, Inline Hooking, …).

### Phase 5: Offensive Tradecraft and Evasion
- EDR Evasion: Learning how modern security tools (CrowdStrike, SentinelOne, …) work. Learning techniques like Indirect Syscalls and Manual PE Loading to stay invisible.
- Shellcode Development: Learning how to write position-independent code (PIC) in ASM.
- Reflective Loading: Mastering the ability to execute code in memory without ever touching the hard drive.

### Phase 6: Vulnerability Research
- Fuzzing: Building automated systems to crash software and find bugs.
- Symbolic Execution: Using advanced math and tools like Z3 or Angr to find paths through complex code.
- Exploit Development: Taking a crash and turning it into a "Primitive" (Read/Write/Execute) to gain full control of a system.



I know that this roadmap is a multi-year plan, I'm totally ready to spend all the years it needs to fully finish! And not all plans are perfect to follow forever, so I might edit this roadmap later when I reach a higher level, goals change, or any other valid reason!
