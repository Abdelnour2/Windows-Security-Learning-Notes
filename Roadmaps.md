# All Roadmaps:
## Roadmap V3 (Current):
- **Courses:**
    - **HTB’s SOC Analyst Path:** Prioritizing the following Modules:
        - Windows Event Logs & Finding Evil
        - Understanding Log Sources & Investigating with Splunk
        - Windows Attacks & Defenses
        - YARA & Sigma for SOC Analysts
        - Detecting Windows Attacks With Splunk
- **Tools:**
    - **SIEM:** Splunk (Main), Elastic (Secondary)
    - **Detection Engines:** Sigma and YARA.
    - **Sysinternals Suite:** Focusing on Sysmon, Process Explorer, and Process Monitor
- **Languages:**
    - C
    - PowerShell
    - Python
    - And maybe Rust and Go to stay up with modern malware
- **OS:**
    - Windows Internals
    - Win32 API
    - **Windows Telemetry Layers:** Learning how Windows generates visibility data, with focusing on Event Tracing for Windows (ETW), and native security event logs
    - Windows Kernel

## Roadmap V2:
- **Courses:**  
    - HTB’s SOC Analyst Job Path
- **Tools:**  
    - **SIEM:** Splunk (main), Elastic (secondary)
    - Microsoft Sysinternals Suite tools like Sysmon, Process Explorer and Process Monitor
- **Languages:**  
    - Python
    - PowerShell
    - C
- **OS Knowledge:**  
    - Windows Internals
    - Win32 API
- **Extra:**  
    - **Cloud Logging:** AWS CloudTrail, Azure Activity Log
    - Projects and Automation
    - Communication and Writing Skills


## Roadmap V1:
**Phase 1:** The Foundations
- **C Programming:** Learning C with focus on memory management, pointers, structures, and how the stack and heap actually work.
- **x86_64 Assembly:** Learning Intel syntax. Understanding registers (RAX, RBX, …), instruction sets, and the calling conventions.
- **Computer Architecture:** Understanding how the CPU interacts with RAM. Learning about the Fetch-Decode-Execute cycle and the difference between User Mode (Ring 3) and Kernel Mode (Ring 0).

**Phase 2:** Windows Internals and User Mode
- **PE File Format:** Understanding the Portable Executable Format (Headers, Sections, IAT, EAT, …) fully.
- **Win32 API:** Learning to interact with Windows directly. Focusing on Process, Thread, Memory, and Handle management.
- **Virtual Memory:** Learning about Page Tables, VAD (Virtual Address Descriptors), and how Windows isolates one process from another.
- **Tools:** x64dbg for dynamic debugging and PE-Bear for static file analysis.

**Phase 3:** Reverse Engineering
- **Static Analysis:** Master Ghidra or IDA Pro. Learning how to identify code patterns (if/else, loops, switch cases, …) in a decompiler.
- **Cracking Logic:** Practicing “Crackmes”. Learning how to bypass license checks, find hardcoded keys, and patch binaries to change their behavior.
- **Anti-Debugging:** Learning how software tries to hide from debuggers (like using IsDebuggerPresent, Timing Checks, …) and how to defeat those protections.

**Phase 4:** The Kernel & Driver Development
- **Windows Driver Model (WDM/KMDF):** Learning to write basic drivers. Understanding the I/O Request Packet (IRP) flow and how drivers communicate with User-Mode apps.
- **Kernel Debugging:** Learning how to set-up dual machines (test machine, and main machine) to deploy the driver on the test machine and test it using a debugger (WinDbg) on the main machine.
- **Hooking Theory:** Learning how to intercept system calls (SSDT Hooking, Inline Hooking, …).

**Phase 5:** Offensive Tradecraft and Evasion
- **EDR Evasion:** Learning how modern security tools (CrowdStrike, SentinelOne, …) work. Learning techniques like Indirect Syscalls and Manual PE Loading to stay invisible.
- **Shellcode Development:** Learning how to write position-independent code (PIC) in ASM.
- **Reflective Loading:** Mastering the ability to execute code in memory without ever touching the hard drive.

**Phase 6:** Vulnerability Research
- **Fuzzing:** Building automated systems to crash software and find bugs.
- **Symbolic Execution:** Using advanced math and tools like Z3 or Angr to find paths through complex code.
- **Exploit Development:** Taking a crash and turning it into a "Primitive" (Read/Write/Execute) to gain full control of a system.