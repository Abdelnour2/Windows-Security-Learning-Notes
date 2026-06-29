# Roadmap V3 - Documentation 7: Monday 29 June 2026
## Summary:
Executed Week 1's Main Iteration of the Detection Engineering Project which was about LSASS Memory Dumping attacks!

## Documentation:
Week 1 of the 5 week process has started! And for the main iteration, I'm going to study, apply, and detect LSASS Memory Dumping attacks!

First, I spent the time researching about the attack!

I've learned that LSASS or Local Security Authority Subsystem Service, is a background process in Windows called lsass.exe. It runs under the highest OS privilege "NT AUTHORITY\SYSTEM". And it can do multiple things like:
- **User Authentication:** Validates local and domain logins when a user inputs a password, smartcard, or biometric scan.
- **Single Sign-On (SSO):** Caches credential structures in memory so users are not repeatedly prompted for passwords when accessing internal network shares, printers, databases, ...
- **Security Token Generation:** Issues access tokens that dictate a user's permissions and group memberships across the system.
- **Policy Enforcement:** Manages local security rules like password complexity, account lockouts, and auditing parameters.

LSASS Memory Dumping attack happens when an attacker copies everything currently sitting inside that memory into a file like a .dmp file. Then the attacker can move the file into their machine to crack it using offline cracking tools to steal cleartext passwords, Windows password hashes, and active login tokens.

And this attack doesn't require specialized tools since it can be done by using built-in Windows tools! It can literally be done by executing 2 lines of PowerShell:
```PowerShell
$LsassPid = (Get-Process lsass).Id
rundll32.exe C:\windows\System32\comsvcs.dll, MiniDump $LsassPid C:\Users\Public\lsass.dmp full
```

In the first line, we call Get-Process to find the lsass process. Once we have it, we take its ID and assign it to the LsassPid variable.

In the second line, we are running comsvcs.dll which is a system DLL that contains administrative functions meant for troubleshooting system crashes! One of these functions which is actually undocumented, is MiniDump. When called, it copies the memory of any process we point it at. It has been primarily created for admins to create a crash dump for debugging purposes, but attackers use it to dump lsass's memory.

Funny enough, testing the script activated Windows Defender, where it classified the script as a Trojand with Severe alert level! Of course this is an excellent thing, but for the lab VM, I'll turn it off and try again!

Trying again also didn't work! Te script ran, but there was no output in the Public folder. After searching, Google AI told me that there is a protection on LSASS called Protected Process Light. Its value was 2 which means heavily protected, so I disabled it by setting the value to 0. Then restarted the VM and tried again. It worked now!

Since the attack has worked, now it's time to make the detections!

First, YARA rules! I'm thinking of searching for all these words:
- rundll32.exe
- lsass.exe
- MiniDump
- comsvcs

And for the condition, I think its best to do all of them since the combination of these is what makes the script dangerous. But if one of them was executed, I think it's normal since as I mentioned, these are all legitimate Windows tools used by admins! I don't know if calling lsass is suspicious in any case so I'll see about it.. but for V1 of the YARA rule, I think I'm going to do all of them!

Rule V1:
```yar
rule LSASS_Memory_Dumping_Attack {
	strings:
		$w1 = "lsass.exe"
		$w2 = "rundll32.exe"
		$w3 = "comsvcs.dll"
		$w4 = "MiniDump"
	condition:
		all of them
}
```

running this rule on the script didn't catch it! So I switched all to any to see what is failing, and I noticed that lsass.exe is not being catched! After reviewing the script, I remembered that the script had lsass and not lsass.exe. So I fixed the rule and it worked! And I added the ascii and wide keywords that I mentioned before.

Rule V2:
```yar
rule LSASS_Memory_Dumping_Attack {
	strings:
		$w1 = "lsass" ascii wide
		$w2 = "rundll32.exe" ascii wide
		$w3 = "comsvcs.dll" ascii wide
		$w4 = "MiniDump" ascii wide
	condition:
		all of them
}
```

I gave the rule to Gemini to give me feedback on it! And it told me that it is good but can be immproved by adding "nocase" keyword too to make the rule non-case sensitive. Also, it added a meta section to give general information about the rule. So I'm going to do that too with all my rules from now on!

Rule V3:
```yar
rule LSASS_Memory_Dumping_Attack {
	meta:
		description = "Detects LSASS Memory Dumping Attacks"	
		date = "28-06-2026"
		severity = "High"

	strings:
		$w1 = "lsass" nocase ascii wide
		$w2 = "rundll32.exe" nocase ascii wide
		$w3 = "comsvcs.dll" nocase ascii wide
		$w4 = "MiniDump" nocase ascii wide
	condition:
		all of them
}
```

Next, Sigma Rules. I simulated the attack again to see what logs are generated on Splunk! There were 2 logs:

The first, was Event ID 1 associated with rundll32.exe indicating its prcoess creation. The event also gave this info:
```xml
<Data Name='CommandLine'>"C:\WINDOWS\system32\rundll32.exe" C:\windows\System32\comsvcs.dll MiniDump 888 C:\Users\Public\lsass.dmp full</Data>
```
Indicating the command that was used to launch rundll32.exe!

I think I can make a Sigma Rule to detect this command? Similar to how I did it with the YARA rule. I believe that I can grab the CommandLine parameter and make a rule out of it! The rule is like this:
```yml
title: LSASS Memory Dumping via Rundll32.exe Command Line
description: Detects process creation of rundll32.exe running comsvcs.dll and MiniDump
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 1
        Image|endswith: "\\rundll32.exe"
        CommandLine|contains|all:
            - "comsvcs.dll"
            - "MiniDump"
    condition: selection
level: high
```

The second event was an Event ID 11 associated with the creation of lsass.dmp. It contained these tags:
```xml
<Data Name='Image'>C:\WINDOWS\system32\rundll32.exe</Data><Data Name='TargetFilename'>C:\Users\Public\lsass.dmp</Data>
```

To make a detection for this, I'm thinking of assigning the Image with rundll32.exe since it is the one creating that file. And here's the tricky part, I can use the TargetFileName to search for anything that ends with lsass.dmp, but what if the attacker named the file differently? can I search for an extension? if I can, then I think it would be best if I searched for *.dmp right? I'll research about it!

I found that indeed I can do that! But no need for the * just .dmp works! So here's my second Sigma rule:
```yml
title: LSASS Memory Dump File Creation via Rundll32
description: Detects the creation of any process memory dump file (.dmp) generated by rundll32.exe
logsource:
  product: windows
  service: sysmon
detection:
  selection:
    EventID: 11
    Image|endswith: "\\rundll32.exe"
    TargetFileName|endswith: ".dmp"
  condition: selection
level: high
```

**Reflection:**  
I really enjoyed executing this iteration! And I learned tons of new information regarding attacks, Windows components, PowerShell scripting, ... And practiced Sigma and YARA rules! I feel that I started to get the hang out of Detection Engineering! And I'll do my best to improve and make more robust detections in the future!