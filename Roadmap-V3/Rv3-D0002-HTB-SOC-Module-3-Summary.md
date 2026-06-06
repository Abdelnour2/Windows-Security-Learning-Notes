# Roadmap V3 - Day 2: Saturday 6 June 2026
## Summary:
Summarized what I learned in HTB's SOC Analyst course's Module 3.

## Documentation:
### Module 3: Windows Event Logs & Finding Evil
This module covers the core mechanics of Windows Event Logs, advanced system telemetry through System Monitor (Sysmon), and kernel-level visibility via Event Tracing for Windows (ETW). It details how to leverage these data sources alongside PowerShell's Get-WinEvent cmdlet to hunt for sophisticated adversary methodologies.

#### **Section 1:** Windows Event Logs Fundamentals
Windows Event Logs are native operating system records that capture structural data from system components, applications, services, and Event Tracing for Windows (ETW) providers.

**Architecture and Default Categories:**  
Logs are stored natively as structured .evtx files and are accessible administratively via the Windows Event Viewer GUI or programmatically via the Windows Event Log API.
- **Application:** Records application-specific errors, warnings, and software diagnostic events.
- **Security:** Tracks authentication, logon sessions, rights management, and object access.
- **Setup:** Logs system installation, updates, and OS configuration activities.
- **System:** Captures internal OS-level engine events, hardware drivers, and core service changes.

**Structural Anatomy of an Event Entry:**  
Every event log contains explicit metadata under the hood, structured and stored in XML format:
- **Event ID:** A unique numerical identifier defining the specific type of occurrence.
- **Level:** Severity classification (Information, Warning, Error, Critical, Verbose).
- **Keywords:** Structural flags (such as Audit Success or Audit Failure) used for rapid search indexing.
- **XML Extended Telemetry:** Contains fine-grained details not always visible in the basic GUI description, such as the exact Process ID (PID) or Logon ID used to chain related events together into an attack timeline.

#### **Section 2:** High-Value Windows Event IDs
Monitoring specific Event IDs enables defenders to isolate signs of defense evasion, authentication abuse, persistence, and lateral movement.

**1. System Infrastructure & Tampering Logs:**
- **ID 1074 (System Shutdown/Restart):** Tracks when and why a system rebooted. Unexpected entries can signal unauthorized administrative access or malware impact.
- **ID 6005 & 6006 (Event Log Service State):** ID 6005 marks the system boot-up sequence. ID 6006 records the intentional stopping of the logging engine; unexpected occurrences suggest an attacker is disabling logging to cover their tracks.
- **ID 6013 (Windows Uptime):** Logged daily to show system uptime in seconds. A sudden shift to a small uptime window indicates an unrecorded or forced reboot.
- **ID 7040 (Service Status Change):** Logs changes to a service's startup configuration (e.g., changing from Disabled to Automatic). This is a primary indicator of system tampering or defense disruption.

**2. Defense Evasion and Anti-Malware Logs:**
- **ID 1102 (The Audit Log Was Cleared):** A high-fidelity indicator of malicious anti-forensics / tampering.
- **ID 1116 (Antimalware Detection):** Microsoft Defender flagged a malicious presence.
- **ID 1118 / 1119 / 1120 (Defender Remediation State):** Tracks the lifecycle of automated remediation (Started / Succeeded / Failed). Failures require immediate manual intervention.
- **ID 4719 (System Audit Policy Changed):** Indicates that an operator modified global auditing rules, often to turn off auditing for specific actions to mask a compromise.
- **ID 4907 (Audit Policy Change / SACL Modification):** Triggered when the System Access Control List (SACL) of a secure object (like a registry key or file path) is modified. Threat actors alter SACLs to stop the OS from generating security alerts on files they intend to tamper with.
- **ID 5001 (Real-Time Protection Configuration Changed):** Generated when Defender's live protection settings are altered or disabled.

**3. Authentication & Account Changes:**
- **ID 4624 / 4625 (Successful / Failed Logon):** Tracks session creation. ID 4625 clusters indicate brute-force or credential stuffing attacks. Evaluation of the Logon Type (e.g., Type 3 Network Logons vs. Type 2 Interactive Logons) is required to determine the context.
- **ID 4648 (Logon with Explicit Credentials):** Triggered when a process attempts to authenticate as a different user explicitly, which is common during lateral movement or privilege escalation.
- **ID 4672 (Special Privileges Assigned):** Generated immediately after a successful logon if administrative or super-user tokens are assigned. Critical Privilege Constants like SeDebugPrivilege must be heavily scrutinized, as they grant the power to read or modify memory belonging to other processes.
- **ID 4738 (User Account Changed):** Tracks modifications to account parameters, group memberships, or password settings, signaling a potential account takeover.
- **ID 4771 (Kerberos Pre-Authentication Failure):** A specific indicator of Kerberos-targeted brute-force or roasting attacks.

**4. Persistence, Object Access & Network Shares:**  
- **ID 4656 (Handle Request to an Object):** Tracks requests to open handles to restricted objects (files, registry keys, or process memory). Used to trace unauthorized asset manipulation.
- **ID 4698 / 4702 (Scheduled Task Created / Updated):** A highly common administrative mechanism abused by malware to establish persistence.
- **ID 7045 (New Service Installed):** Generated when a new service registration occurs in the system, a premier persistence vector for backdoors.
- **ID 5140 / 5142 / 5145 (Network Share Activity):** Tracks when network shares are accessed, added, or enumerated for access rights. Frequent checks often indicate an attacker mapping the network or preparing to exfil data.
- **ID 5157 (Windows Filtering Platform Block):** Logs when the native host firewall blocks an outbound or inbound connection attempt. Useful for tracking blocked command-and-control (C2) beacons.

#### **Section 3:** Advanced Telemetry via System Monitor (Sysmon)
System Monitor (Sysmon) is a persistent Windows service and device driver that runs in the background to capture deep, low-level system events that are typically absent from standard security event logs.

**Configuration Controls:**  
Sysmon relies on an XML-based configuration file to manage event inclusion or exclusion rules, filtering out standard environmental noise while preserving high-fidelity threat indicators.
- **sysmon-config (SwiftOnSecurity):** A widely adopted baseline optimized for generic enterprise environments.
- **sysmon-modular (Olaf Hartong):** A highly customizable, modular approach suited for tailored detection engineering.

**Administrative CommandsL**  
From an elevated Command Prompt, Sysmon can be deployed or reconfigured using the following parameters:
- **Initial Deployment:** Installs the driver, sets process hashing algorithms (MD5/SHA256/Imphash), enables module load tracking (-l), and initiates network monitoring (-n).
    - sysmon.exe -i -accepteula -h md5,sha256,imphash -l -n
- **Applying/Updating Configuration:** Dynamically updates active filtering logic without restarting the service.
    - sysmon.exe -c filename.xml

#### **Section 4:** Sysmon Threat Hunting Case Studies
**1. Detecting DLL Hijacking:**
- **Telemetry Target:** Sysmon Event ID 7 (Image/Module Load). Tracks when a process loads a DLL, capturing the full file path and digital signature status.
- **Attack Vector:** An attacker places a malicious DLL with a legitimate name into a user-writable directory alongside a trusted executable, forcing the executable to load the malicious payload instead of the authentic system DLL located in System32.
- **Core Detection Logic:**
    - **Path Mismatch:** A trusted system binary executes out of a non-standard, user-writable directory (e.g., \Downloads\ or \Desktop\\).
    - **System DLL Misplacement:** A core system DLL is loaded from a path outside of C:\Windows\System32\ or C:\Windows\SysWOW64\\.
    - **Signature Anomaly:** The loaded module is unsigned, whereas legitimate Windows core DLLs carry verified Microsoft cryptographic signatures.

**2. Detecting Unmanaged PowerShell / C# Code Injection:**
- **Telemetry Target:** Sysmon Event ID 7 (Image/Module Load).
- **Attack Vector:** Attackers inject C# execution code capability directly into native, unmanaged Windows system processes to execute tools completely in-memory, bypassing standard disk-based scripts detection.
- **Core Detection Logic:** Look for the loading of the Microsoft .NET Common Language Runtime (CLR) libraries, specifically clr.dll and clrjit.dll, inside native Windows processes that have no functional reason to interact with the .NET framework (e.g., spoolsv.exe, notepad.exe, svchost.exe). A process transitioning from an unmanaged baseline state to a managed (.NET) execution state is a high-fidelity indicator of code injection.

**3. Detecting Credential Dumping (LSASS Memory Targeting):**
- **Telemetry Target:** Sysmon Event ID 10 (ProcessAccess). Monitors when a source process requests an open handle to read or manipulate the memory space of a target process.
- **Attack Vector:** Tools like Mimikatz open a handle to the Local Security Authority Subsystem Service (lsass.exe) to extract plaintext credentials or active NTLM password hashes from memory.
- **Core Detection Logic:**
    - **Untrusted Source Paths:** Non-standard binaries originating from writable directories requesting handles to lsass.exe.
    - **User Context Mismatch:** The SourceUser context initiating the handle request differs from the TargetUser context (e.g., a standard user context attempting to query a process running as NT AUTHORITY\SYSTEM).
    - **Privilege Abuse:** Flag processes requesting the activation of SeDebugPrivilege outside of verified administrative or developer debugging tools.

#### **Section 5:** Event Tracing for Windows (ETW)
Event Tracing for Windows (ETW) is a high-speed, kernel-level tracing facility built directly into the Windows operating system. It relies on a lightweight publish-subscribe buffering model to capture granular telemetry generated by user-mode applications and kernel-mode device drivers.

**Architectural Core Components:**  
- **Controllers:** Manage the lifecycle of tracing sessions; they start/stop trace collections and subscribe to individual providers using native tools like logman.exe.
- **Providers (Publishers):** Internal OS or third-party components that generate telemetry events. They remain dormant by default to preserve system performance, initializing only when activated by a controller.
- **Consumers (Subscribers):** Defensive software, SIEM agents, or analytical scripts that hook into active sessions to process and read streaming events.
- **Channels:** Logical routing pipelines used to organize and direct events. An ETW event must have a distinct Channel property applied to it; otherwise, it cannot be processed or consumed by the Windows Event Log engine.
- **ETL Files (.etl):** Durable, raw storage containers where ETW writes real-time binary data to disk, allowing for later offline forensic analysis and historical log rotation.

**High-Value Native ETW Providers:**  
The following providers supply deep system context that bypasses typical user-mode logging limitations:
| Provder Name | Key Detection Scenarios & Use Cases |
|:------------:|:-----------------------------------:|
| Microsoft-Windows-Kernel-Process | Exposes process hollowing, Parent PID (PPID) spoofing, and abnormal process lifecycles directly from the kernel |
| Microsoft-Windows-Kernel-File | Tracks unauthorized file access, critical system path tampering, and rapid file modifications indicative of ransomware |
| Microsoft-Windows-Kernel-Network | Provides kernel-level tracking of all network socket connections, helping uncover hidden C2 traffic channels |
| Microsoft-Windows-Kernel-Registry | Monitors low-level key/value additions, changes, or deletions, typically tracking malware persistence mechanisms |
| Microsoft-Windows-DotNETRuntime | Captures internal execution namespaces and assembly loading inside the .NET runtime engine, exposing memory-only assembly execution attacks |
| Microsoft-Windows-PowerShell | Delivers script block logging, variable state tracking, and execution context visibility |
| Microsoft-Windows-DNS-Client | Audits explicit lookups to external malicious domains and captures DNS tunneling attempts |
| Microsoft-Windows-CodeIntegrity | Flags attempts to bypass driver signing enforcements or load malicious, unsigned kernel drivers |

**Restricted Providers & The Access Barrier:**  
Certain high-value providers, such as Microsoft-Windows-Threat-Intelligence (WinTI / Symi), provide unparalleled kernel-level visibility into advanced process memory manipulation, remote thread injections, and evasion tactics.

To protect this sensitive stream from being tampered with or disabled by malware, Windows places strict restrictions on access:
- A consuming process must run with Protected Process Light (PPL) privileges.
- Achieving PPL validation requires security vendors to pass rigorous vetting by Microsoft, execute legal compliance frameworks, and deploy a verified Early Launch Anti-Malware (ELAM) driver carrying a specialized Authenticode signature.

**Advanced ETW Detection Use Cases:**
**1. Uncovering Parent PID (PPID) Spoofing**
- **The Technique:** Adversaries leverage user-mode API tokens during process creation to explicitly define a fake parent process (e.g., spawning a malicious cmd.exe but declaring a trusted process like spoolsv.exe as the parent). This easily evades standard parent-child behavioral alerts.
- **The Telemetry Deficit:** Standard process monitoring and basic Sysmon Event ID 1 logs blindly accept the user-mode API parameters. Consequently, Sysmon will incorrectly log the spoofed binary as the true parent.
- **The ETW Solution:** The Microsoft-Windows-Kernel-Process provider records process initialization directly from the kernel engine. Because it monitors the transaction at the kernel layer rather than relying on user-mode API fields, it identifies the true creator process, instantly exposing the spoofing attempt.

**2. Spotting Malicious Memory-Only .NET Assembly Loads:**
- **The Technique:** To bypass traditional disk-based file inspection, threat actors move away from standard scripts and pivot to Bring Your Own Land (BYOL) strategies. They load compiled C# binary assemblies (e.g., Seatbelt) directly into the memory space of an existing process.
- **The Telemetry Deficit:** While checking for Sysmon Event ID 7 can flag the entry of runtime engines (clr.dll) into unusual processes, it is noisy and cannot look inside the engine to see what code actually ran.
- **The ETW Solution:** Hooking into the Microsoft-Windows-DotNETRuntime provider captures the exact internal namespaces, class declarations, and method execution paths running inside the memory space of the runtime engine, rendering the memory-only payload completely visible.

#### **Section 6:** Programmatic Log Queries via PowerShell (Get-WinEvent)
The Get-WinEvent cmdlet is a core tool for querying local or remote Windows Event logs, analyzing offline .evtx files, and parsing live ETW traces.

**1. Essential Baseline Parameters:**
- **-ListLog *:** Pulls a complete index of all system log channels alongside their active properties (e.g., RecordCount, IsEnabled, LogMode).
- **-ListProvider *:** Lists all system-registered event providers and their associated log routing paths.
- **-MaxEvents <Int>:** Restricts the maximum record count returned by the query to prevent terminal flooding and memory strain.
- **-Oldest:** Reverses the default sorting mechanism to return events chronologically (oldest records first).
- **-Path "C:\TargetLogs.evtx":** Directs the query to parse an offline or exported log file, which is an essential requirement for forensic workstation environments.

**2. High-Performance Filtering Strategies:**
**Performance Optimization Rule:** Always apply filtering as close to the data source as possible using -FilterHashtable or -FilterXPath. Avoid piping large, unfiltered event streams into Where-Object, as doing so forces local memory to parse thousands of raw objects, causing severe processing overhead.

**A. FilterHashtable Query:**  
Leverages explicit key-value arrays to isolate events quickly before returning results to the host. Valid keys include LogName, Path, ID, ProviderName, Level, StartTime, and EndTime.

**B. FilterXPath Query:**  
Allows for deep, precise queries directed at the underlying XML architecture of the event log entry. This is highly effective for isolating individual variables hidden deep within the <EventData> blocks, such as specific command-line arguments or remote IP strings.

**C. Pipeline Property Index Filtering:**  
When conducting fine-grained ad-hoc structural parsing via Where-Object, the raw data fields of an entry are stored inside an indexed array called .Properties. Specific parameters can be matched by targeting their exact position in the array.