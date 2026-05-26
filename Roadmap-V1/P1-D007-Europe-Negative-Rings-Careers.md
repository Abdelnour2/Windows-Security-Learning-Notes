# Phase 1 - Day 7: Saturday 4 April 2026
## Summary:
Did a side research on the European situation of ditching American products, then researched the career of working as a Security Engineer / Researcher in the negative privrings rings.

## Note:
There is no Notes, only Documentation.

## Documentation:
I was able to dedicate some time to research today, not much though, so I didn’t start researching the Heap. Instead, I wanted to know more about the European situation in which they're trying to replace all (or almost all) American products with their own! The latest thing I know so far are:
- Released Euro Office, replacing MS Office and Google Workstation
- Working on the Digital Euro, replacing VISA and MasterCard
- Building a massive infrastructure to make their own Cloud and AI.
- Replacing Windows with Linux
- Replacing US Social Media platforms like Meta Apps, MS Teams, Zoom, … with European products.

This is really interesting and got me curious about: If someone with my specialization goal (Windows System Security) wants to work and live in Europe, is this path still valid with all this change?

So I started researching to find an answer to my question. My research resulted with a yes! Specializing in Windows System Security is still a valid option even in Europe for many reasons:
- Windows is still the dominant in the world. So doing security research on it will be valuable whatever the place is.
- Windows is still used (at least for now) in private sectors, so they’ll need Windows Security people.
- MS still has offices in Europe. In addition to adding more data centers in Europe.
- Similar to the first point. Doing security research on an OS can be done from anywhere. Unless you work at a company that requires the research to be done on a different OS.

After that, I went back to the “Security Rings” research I did earlier! I was wondering if this 7 layer thing is only present on Windows or a universal thing. So I researched and found that it is universal and not tied to Operating Systems. Instead, it is a characteristic of the x86/x64 CPU hardware architecture. Rings -3, -2, and -1 exist regardless of which OS is installed since they are embedded in the motherboard and CPU silicon. And most modern Operating Systems (Windows, macOS, and Linux) only use Ring 0 and Ring 3.

Another thing that got me wondering about these layers is the career of the security people working in the negative layers. What’s the position/s called? What do they work on? What do they learn? Is it a good career? Is a career in this better than specializing in just Windows systems?

What I found is that a career in this domain is an extremely niche and very specialized domain. It has positions as Firmware Security, Platform Security, or Low-Level Systems Research. Depending on the position, they can work on multiple stuff like securing the BIOS/UEFI systems at Ring -2. Finding vulnerabilities and securing the hardware itself of Intel/AMD/NVIDIA/Apple Silicon at Ring -3. The way of working requires a lot of reverse engineering, vulnerability research, and defense design and implementation. They work in pure C and ASM, there are no built-in functions that help the process like the Windows API. Since the domain is extremely niche and highly specialized, work locations are much less than Windows security. In addition, this domain is considered for seniors or elite in the security field. In terms of “is it better”, both domains are extremely valuable, in demand, and pay very high salaries. So it depends on the person’s interest.
