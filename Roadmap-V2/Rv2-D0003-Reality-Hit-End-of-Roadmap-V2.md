# Roadmap V2 - Day 3: Friday 5 June 2026
## Summary:
Ending Roadmap V2, and starting V3 after finding out that Malware Analysis is not an intern/entry level job!

## Documentation:
While practicing, Researching, Learning, and going through Module 3 in HTB's SOC Analyst Job Path titled "Windows Event Logs & Finding Evil", I've learned online that Malware Analysis is not an internship/entry level job, instead it is a mid/senior level title. So I started researching to confirm the reality of this sentence, and it turned out to be true, Malware Analysis is extremely rare to be found as an internship/entry level job.

So I spent the whole last two days researching internship/entry level positions that I can target while keeping Malware Analysis as a later position. There were many options! There was of course the famous Tier 1 SOC Analyst path which is one of the most common starting points for security professionals. In addition, there were Cloud Security, General IT, maybe Information Security, and others.

But there were two positions that caught my attention while researching and made me dig into them! These two positions were Detection Engineering, and Endpoint/EDR Security Engineering! And after researching, I chose to go with Detection Engineering!

The reason behind my choice is that Detection Engineering's work is often tied with analyzing Malware reports and writing rules to prevent these malware from happening! While Endpoint/EDR Security Engineering often includes kernel level work. Making Kernel Filter Drivers to catch advanced levels of malware. While I do love going deeper and learning about the Kernel which I already did before, I prefer to stay in Ring 3 for a while to get professional in it before going down, since ~90% of malware runs on Ring 3 anyways. So for now, I’m going to stay in Ring 3, focusing on Detection Engineering which thinking about it also improves my programming and malware knowledge for many reasons:
- I can build my own malware, analyze them, and try to prevent them (improves programming, malware analysis, and Detection Engineering)
- I can read about malware and see how to prevent them (improves Win32 and malware knowledge, and Detection Engineering)

So as I said, I’ll focus on Detection Engineering while at the same time, keeping an eye on the kernel too to not miss out on knowledge!

And of course, since I’m targeting a new tiitle, the roadmap will change too! The new roadmap will be:
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

With that, I’m ending Roadmap V2 here, and starting a new V3 one. And for a better experience, I'm going to reupload Module 1 and 2’s summary there too to not go back and forth between the 2 versions.