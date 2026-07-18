# Roadmap V3 - Documentation 10: Saturday 11 July 2026
## Summary:
Executed Week 3's Main Iteration (which is the 5th iteration in general) of the Detection Engineering Project which was about Scheduled Tasks Attacks!

## Documentation:
Due to a tight schedule, week 2's secondary iteration was pushed back. The goal is to wrap up this week’s iterations quickly so I can circle back and complete Week 2's secondary iteration!

The main iteration of Week 3 is about Scheduled Tasks attacks. Attackers frequently abuse the Windows Task Scheduler to achieve persistence, ensuring their malicious access survives system reboots, user logouts, or password changes. This technique is highly favored because tasks can automatically restart if a malicious process is killed, can elevate privileges by running under high-privilege accounts like NT AUTHORITY\SYSTEM, and easily evade detection by blending into the noise of legitimate software updates. Mechanically, Windows stores these task configurations as XML files in C:\Windows\System32\Tasks\ while mirroring them in the registry under HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree, where the Task Scheduler service (Taskschd.msc) ultimately executes them via the netsvcs shared service host (svchost.exe).

This attack can be simulated by running this PowerShell script:
```ps1
schtasks.exe /create /tn "DE-Project-W3-M" /tr "C:\Users\Public\main.exe" /sc daily /st 12:00
```

Where schtasks.exe is the native Windows command-line tool used to create, delete, query, change, run, and end scheduled tasks on a local or remote computer. Since we want to create a new task I used the /create option, then /tn is used to choose the name of this task. tn is short for Task Name. Then /tr (short for Task Run) is used to specify the full path of the program/script we want to execute in this task! I went for the main.exe program I created all the way back in iteration 1. Then /sc (short for schedule) defines the frequency of this task, I set it to daily which means that this task will be triggered once every day! Finally /st (short for Start Time) is used to set the exact time this task will trigger! I set it to 12 PM!

Now that we have the script, I'll move to making the YARA rule now. This is a bit tricky since this is a native Windows tool used by admins, so I'm going to do my best to catch the combination of the important keywords that indicates an attack! I'll be catching the "schtasks" and "/create", I'm not going to catch anything in the naming since attackers can name this anything! Then for the path I'm going to catch the usual folders like Public and AppData! maybe catch daily too since of course attackers need their attack to be active as much as possible! I'm not going to catch the timing since there are lots of possibilities! So this is what I'm going to do:
"1. Mandatory:"
- "schtasks"
- "/create"
- "Public" or "AppData"

"2. Optional:"
- "daily"

But I immediately noticed a weakness in these conditions while writing them, which is the folder location! attackers can also use other locations too, not only those two! So I'm thinking if I should put them as optional keywords! But I'm also thinking, if only the first two keywords are mandatory, doesn't this lead to a lot of false positives? What I'm going to do right now is to keep "public" and "AppData" as mandatory since we know that they are the most common paths for attackers! And I'll see later about how I can improve my rules!

With that, the rule will be like this:
```yar
rule Scheduled_Tasks_Attack {
	meta:
		description = "Detects if a program/script found in Public/AppData folders is being set as a Scheduled Task"
		date = "11/07/2026"
		severity = "high"
	strings:
		// Mandatory
		$schtasks = "schtasks" ascii wide nocase
		$create = "/create" ascii wide nocase
		
		// Mandatory - 1 of them
		$public_dir = "Public" ascii wide nocase
		$appdata_dir = "AppData" ascii wide nocase

		// Optional
		$daily = "daily" ascii wide nocase
	condition:
		$schtasks and $create and ($public_dir or $appdata_dir or $daily)
}
```

Next, Sigma rules! I ran the script and it generated 6 events, these are the important extracted details from each one:
**Event 1:**  
- `<EventID>1</EventID>`
- `<Data Name='Image'>C:\Windows\System32\schtasks.exe</Data>`
- `<Data Name='Description'>Task Scheduler Configuration Tool</Data>`
- `<Data Name='OriginalFileName'>schtasks.exe</Data>`
- `<Data Name='CommandLine'>"C:\WINDOWS\system32\schtasks.exe" /create /tn DE-Project-W3-M /tr C:\Users\Public\main.exe /sc daily /st 12:00</Data>`
- `<Data Name='CurrentDirectory'>C:\Users\Snow\Desktop\</Data>`
- `<Data Name='ParentImage'>C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe</Data>`
- `<Data Name='ParentCommandLine'>C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe</Data>`

**Event 2:**
- `<EventID>11</EventID>`
- `<Data Name='Image'>C:\WINDOWS\system32\svchost.exe</Data>`
- `<Data Name='TargetFilename'>C:\Windows\System32\Tasks\DE-Project-W3-M</Data>`
- `<Data Name='User'>NT AUTHORITY\SYSTEM</Data>`

**Event 3:**
- `<EventID>13</EventID>`
- `<Data Name='EventType'>SetValue</Data>`
- `<Data Name='Image'>C:\WINDOWS\System32\svchost.exe</Data>`
- `<Data Name='TargetObject'>\REGISTRY\A\{bb10327d-8983-ec7b-46c4-990a33cfd99b}\Root\InventoryApplicationFile\schtasks.exe|ab37f90cb14ae5a6\LowerCaseLongPath</Data>`
- `<Data Name='Details'>c:\windows\system32\schtasks.exe</Data>`
- `<Data Name='User'>NT AUTHORITY\SYSTEM</Data>`

**Event 4:**
- `<EventID>13</EventID>`
- `<Data Name='EventType'>SetValue</Data>`
- `<Data Name='Image'>C:\WINDOWS\System32\svchost.exe</Data>`
- `<Data Name='TargetObject'>\REGISTRY\A\{bb10327d-8983-ec7b-46c4-990a33cfd99b}\Root\InventoryApplicationFile\schtasks.exe|ab37f90cb14ae5a6\Publisher</Data>`
- `<Data Name='User'>NT AUTHORITY\SYSTEM</Data>`

**Event 5:**
- `<EventID>13</EventID>`
- `<Data Name='EventType'>SetValue</Data>`
- `<Data Name='Image'>C:\WINDOWS\System32\svchost.exe</Data>`
- `<Data Name='TargetObject'>\REGISTRY\A\{bb10327d-8983-ec7b-46c4-990a33cfd99b}\Root\InventoryApplicationFile\schtasks.exe|ab37f90cb14ae5a6\BinProductVersion</Data>`
- `<Data Name='User'>NT AUTHORITY\SYSTEM</Data>`

**Event 6:**
- `<EventID>13</EventID>`
- `<Data Name='EventType'>SetValue</Data>`
- `<Data Name='Image'>C:\WINDOWS\System32\svchost.exe</Data>`
- `<Data Name='TargetObject'>\REGISTRY\A\{bb10327d-8983-ec7b-46c4-990a33cfd99b}\Root\InventoryApplicationFile\schtasks.exe|ab37f90cb14ae5a6\LinkDate</Data>`
- `<Data Name='User'>NT AUTHORITY\SYSTEM</Data>`

Personally I believe that only the first event is worth making a rule for since the other 5 events are purely informational that doesn't provide explicit details into the attack!

So for this Sigma rule, I'm going to detect Event ID 1 where the original file name is schtasks.exe, and the parent image contains powershell.exe and the command line contains "/create" "Public" or "AppData", and optionally "daily"!

```yml
title: schtasks.exe Spawned by PowerShell
description: Detects when PowerShell creates a schtasks process with a command line that contains "/create", "Public" or "AppData", and optionally "daily"
date: 2026/07/11
logsource:
  product: windows
  service: sysmon
detection:
  general:
    EventID: 1
    OriginalFileName: 'schtasks.exe'
    ParentImage|endswith: '\powershell.exe'
  commandLine_mandatory_keywords:
    CommandLine|contains|all|nocase:
      - 'schtasks'
      - '/create'
  commandLine_mandatory_paths:
    CommandLine|contains|nocase:
      - 'Public'
      - 'AppData'
  commandLine_optional:
    CommandLine|contains: 'daily'
  condition: general and commandLine_mandatory_keywords and commandLine_mandatory_paths
level: high
```

I think that Event 2 can also be used to make a rule but only if I had a list of approved task names to match for, I can make a rule where if the name of the file doesn't match this list, then alert! But since I don't have a list, I'll skip it!

With that, this iteration is complete!

**Reflection:**  
This was another very enjoyable iteration to make, learned a new attack from it, and practiced my technical and analytical skills!