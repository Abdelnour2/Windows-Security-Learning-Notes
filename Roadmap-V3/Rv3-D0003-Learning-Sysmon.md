# Roadmap V3 - Day 3: Tuesday 9 June 2026
## Summary:
Deep-Dived into Sysmon

## Documentation:
Today, I've decided to dive into Sysmon and familiarize myself with it since I believe it is one of the main tools used in Detection Engineering, and Malware Analysis. It was even mentioned in the “Windows Event Logs & Finding Evil” module on HTB that I finished a couple of days ago, so I’ll revisit that module too.

This documentation will be a casual one documenting how I’ll go through Sysmon step by step!

First, I’ll read Microsoft’s documentation on Sysmon found on this link: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon and document the key learnings while doing so.

From the introduction, we know that Sysmon (short for System Monitor) is a Windows system service and a device driver that stays active all the time even after rebooting! It monitors and logs everything happening on the system in detail. In addition to that it doesn’t hide itself on the system, so an attacker can know that Sysmon is active. And it doesn’t analyze the data, it just collects them!

**Sysmon Usage:**
- **Install:** sysmon64 -i [\<configfile>]
    - **-i:** Installs service and driver
    - The configuration file is optional
- **Update configuration:** sysmon64 -c [\<configfile>]
    - **-c:** Update the configuration if the configuration file was provided. Prints the current configurations if the configuration was not provided.
    - The configuration file is optional
    - **-c --:** Adding -- after -c resets Sysmon’s configurations to the default one
- **Install event manifest:** sysmon64 -m
    - **-m:** Register sysmon’s event blueprint in Windows so the Event’s Viewer can translate raw log data into human-readable text.
- **Print schema:** sysmon64 -s
    - **-s:** Prints every single possible rule and field Sysmon support.
- **Uninstall:** sysmon64 -u [force]
    - **-u:** Uninstall service and driver.
    - **-u force:** Causes uninstall to proceed even when some components are not installed.

Events are stored in “Applications and Services Logs” under “/Microsoft/Windows/Sysmon/Operational”. And they are in UTC standard time.

Adding -accepteula before -i command skips the End User License Agreement (EULA) popup, it makes Sysmon accept them automatically and then install.

**Sysmon Events:**  
There are 30 events, I’ll explain each one of them as clear as I can:
- **Event ID 1 - Process Creation:** This event triggers whenever a process is created! This could be by double-clicking an executable, or launching a program from the command line. The data it provides with it include:
    - **CommandLine:** The command that created the new process
    - **ParentImage:** Full path of the program used to create that process (Like Explorer, or CMD).
    - **ParentCommandLine:** The command that started the program that started the new process.
- **Event ID 2 - A Process Changed a File Creation Time:** This event triggers when a program manually changes the creation time of a file.
- **Event ID 3 - Network Connection:** This event tracks all inbound and outbound network traffic (TCP and UDP), and it is disabled by default. It connects every network connection directly to the specific program that made it, and it records both source and destination details.
- **Event ID 4 - Sysmon Service State Change:* It basically logs whenever the Sysmon service itself is turned on or off.
- **Event ID 5 - Process Terminated:** Basically logs exactly when a process is closed or forced to stop.
- **Event ID 6 - Driver Loaded:** It logs whenever a device driver is loaded into the Windows kernel.
- **Event ID 7 - Image Loaded:** Logs whenever a process loads a module like a DLL or executable code into its running memory. This is disabled by default. The data collected here are the host process, the specific modules loaded, its file hashes, and its digital status. And enabling this event needs to be done carefully since it can flood the logs list instantly.
- **Event ID 8 - CreateRemoteThread:** Logs when a process creates a thread in another process. Legitimate apps rarely do this so seeing this event mostly indicates malware. The data captured by this event includes the source process (the injector), the target process, and the code fingerprint. Regarding the code fingerprint, it captures exactly where the code starts executing via StartAddress, StartModule (the file name - like a DLL), and StartFunction (the specific action). StartModule and StartFunction can be seen empty if the attacker injects raw shellcode (ShellCode Injection) directly into empty memory rather than loading a standard file module (DLL Injection). Seeing these fields empty is a huge indicator of an attack.
- **Event ID 9 - RawAccessRead:** Detects when a process is reading data directly from the raw drive. Data stored by the event include the process doing the reading and the target device (the hard drive or a specific partition)
- **Event ID 10 - ProcessAccess:** Logs when a process is trying to open and have access to another process. Data stored in this event include the original process (that is trying to get access), the target process, and the specific access rights requested.
- **Event ID 11 - FileCreate:** Logs whenever a file is created or overwritten on the hard drive. Data stored in this event include the program that created the file and the full path of the new file. This event needs to be configured carefully or it will generate a large amount of logs.
- **Event ID 12 - RegistryEvent (Object Create or Delete):** Logs whenever a Windows Registry key or value is created or deleted. The names of the registry root keys are shortened to save space and memory:
    - **HKEY_LOCAL_MACHINE (Stores settings/configurations for the entire machine):** HKLM
    - **HKEY_USERS (Stores settings/configuration for each specific user):** HKU
    - **HKEY_LOCAL_MACHINE\System\ControlSet00x (Core system startup configurations and backups of them):** HKLM\System\CurrentControlSet
    - **HKEY_LOCAL_MACHINE\Classes (Handles file associations):** HKCR
- **Event ID 13 - RegistryEvent (Value Set):** Logs whenever a program changes or modifies an existing Registry value.
- **Event ID 14 - RegistryEvent (Key and Value Rename):** Logs whenever a program renames an existing Registry key or value.
- **Event ID 15 - FileCreateStreamHash:** Tracks the creation of Alternate Data Streams (ADS), which is the primary way Windows flags files downloaded from the internet.
- **Event ID 16 - ServiceConfigurationChange:** Logs any changes to the Sysmon configuration file or its filtering rules.
- **Event ID 17 - PipeEvent (Pipe Created):** Logs the creation of a Named Pipe on the system. A Named Pipe is a communication tunnel that programs use to communicate/share data with each other. For example Google Chrome might create a pipe to let opened tabs (which are separate processes) connect to it and send their data to Chrome to display the content.
- **Event ID 18 - PipeEvent (Pipe Connected):** Logs the exact moment a second process connects to a Named Pipe.
- **Event ID 19 - WmiEvent (WmiEventFilter Activity Detected):** Windows Management Instrumentation (WMI) is a built-in system administration tool. Attackers abuse it to create hidden, fileless timers that run their malware automatically. Event 19 logs the registration of a query that monitors the OS for a specific state or event.
- **Event ID 20 - WmiEvent (WmiEventConsumer Activity Detected):** Logs the registration of the specific task/script that will run when a condition is met.
- **Event ID 21 - WmiEvent (WmiEventConsumerToFilter Activity Detected):** Logs the registration of the link that connects a specific filter condition (logged in Event 19) directly to a consumer action (Logged in Event 20).
- **Event ID 22 - DNSEvent (DNS Query):** Logs every single DNS lookup request made by the computer.
- **Event ID 23 - FileDelete (File Delete Archived):** Logs the deletion of a file and before the file is deleted, Sysmon copies that file and stores it in an archive folder. Because Sysmon copies the files, this might fill a computer’s storage in the long run.
- **Event ID 24 - ClipboardChange (New Content in the Clipboard):** Logs whenever a user or a program copies new text or data into the Windows clipboard.
- **Event ID 25 - ProcessTampering (Process Image Change):** Triggered whenever a change in a process’s executable code happens right as it launches.
- **Event ID 26 - FileDeleteDetected (File Delete Logged):** Logs when a file is deleted without copying it. This is often used as an alternative to Event ID 23 to save space.
- **Event ID 27 - FileBlockExecutable:** Logs the exact moment Sysmon blocks a program from saving a new executable file onto the hard drive.
- **Event ID 28 - FileBlockShredding:** Logs the moment Sysmon blocks a program from completely destroying and shredding a file on the disk.
- **Event ID 29 - FileExecutableDetected:** Logs the moment a new executable file is successfully created on the system, without blocking it.
- **Event ID 255 - Error:** Logs internal errors, failures, or critical bugs occurring within the Sysmon service itself.

**Filters:**  
We can specify in the configuration file what to log, what to skip, and what specific thing to log in a certain event by using Filtering Entries.

**1. Onmatch:**  
The first filter is “onmatch”, it can hold a value of either “include” or “exclude”.
- **onmatch=“include”:** Blocks all event tracking by default. It only logs items that explicitly match our defined criteria.
- **onmatch=“exclude”:** Logs all event tracking by default. It skips items that match our criteria.
If a single event type uses both include and exclude filters, exclude always win.

**2. Logic and Rule Groups:**
- **OR Logic:** using the same field name evaluate as OR conditions. Example: matching either a.exe or b.exe.
- **AND Logic:** Rules using different field names evaluate as AND conditions. All criteria must be met at the same moment.

**3. Advanced Matching Attributes:**
- **Text Conditions:** Fields use case-insensitive attributes like “contains”, “begin with”, “end with” and “is not” to precisely scan text strings without exact path matches.
- **Mini–Value Arrays:** Conditions like “contains any” or “is any” allow to separate multiple strings using a “;” inside a single line of XML code.
- **Smart Image Condition:** The “condition=“image”” attribute allows us to type a simple program name and Sysmon will automatically map and match its full system path.
- **Rule Naming:** We can append a “name=“”” attribute to any rule. Sysmon will stamp this custom label directly into the final Event Log.

Next, I went to YouTube to see if there’re any practical explanations for Sysmon and I found this video from TCM: https://www.youtube.com/watch?v=OAuVYbn1m3A
It was a really good video demonstrating installation, choosing a configuration, demonstrating an attack chain, then reading the logs to analyze and piece out that attack!

**Tomorrow's Plan:**  
I'll be learning Splunk, and learning how to write dectections combining Sysmon with SPL!