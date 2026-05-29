# Roadmap V2 - Day 1: Friday 29 May 2026
## Summary:
Summarized what I learned in HTB's SOC Analyst course's Module 1

## Documentation:
### Module 1: Incident Handling Process
This module covers the foundational principles of Incident Response (IR), the lifecycle of an attack via the Cyber Kill Chain, mapped threat intelligence frameworks, and a deep dive into the four core phases of the NIST Incident Handling Process.

#### **Section 1:** Incident Handling Fundamentals
An organization must maintain a robust incident handling capability to protect its data assets. This capability can be hosted completely in-house, or outsourced to third-party providers (MSSP/MDR) for continuous monitoring or on-demand escalation.

**Terms:**
- **Event:** Any observable occurrence in a system or network (e.g., a user sending an email, a firewall allowing a connection, or a mouse click).
- **Incident:** An event that results in negative consequences (e.g., a system crash).
- **IT Security Incident:** A malicious event explicitly intended to compromise the CIA Triad of an asset or network
    - **Confidentiality:** Preventing unauthorized access to data.
    - **Integrity:** Preventing unauthorized modification of data.
    - **Availability:** Ensuring systems and services remain accessible.

**Common Incident Categories:**
- Data or funds theft
- Unauthorized data access / Rogue remote access
- Malware infections
- Malicious insiders
- System availability issues
- Loss of intellectual property (IP)

**Incident Prioritization:**  
Because resources are finite, incidents must be systematically prioritized based on:
- Severity and level of sophistication
- Business impact and disruption.
- Number of affected systems.
- Sensitivity of compromised data.

The incident handling team is led by an Incident Manager. This role is typically assumed by a SOC Manager, CISO, CIO, or a trusted third-party vendor retainer. A primary reference framework for this execution is the NIST SP 800-61 (Computer Security Incident Handling Guide).

<br>

#### **Section 2:** Cyber Attack Lifecycles and Frameworks

**The Cyber Kill Chain:**  
A 7-stage attack lifecycle, it describes how an adversary executes an objective. In practice, these phases can be fluid, non-linear, or repeated.
1. **Reconnaissance:** Target selection and information gathering. It can be done in two ways:
    - **Passive:** OSINT via LinkedIn, corporate job postings (can reveal internal tech stacks, OS, and AV solutions), and public documentations.
    - **Active:** Network infrastructure scanning, port mapping, vulnerability identification.
2. **Weaponization:** Developing or pairing a lightweight, undetectable malware payload/backdoor with an exploit. The payload is engineered to survive reboots (persistance) and downlaod extra modular tools on demand.
3. **Delivery:** Transmission of the payload to the victim. Common vectors include:
    - **Phishing/Vishing:** Email attachements, malicious links, or phone-based social engineering.
    - **Watering Holes/Spoofed Sites:** Cloned corporate portals or malicious web pages hosting browser exploits
    - **Physical Media:** Rogue USB drops within target facilities.
4. **Exploitation:** The delivered payload is triggered (often requiring just a single user action, like double-clicking an attachment), executing code to gain control of the target process.
5. **Installation:** Implementing a persistent foothold
    - **Droppers:** Small code footprints that pull down the heavier primary malware.
    - **Backdoors/Rootkits:** Highly stealthy mechanisms designed to hide from AV tools and grant persistent system access.
6. **Command and Control (C2):** Establishing an inbound remote channel to an external infrastructure. Advanced threat groups deploy multiple disparate variants of C2 tools to preserve access if one variant is discovered and contained.
7. **Actions on Objectives:** Executing the ultimate goal, which ranges from data exfiltration to deploying network-wide ransomeware.

**MITRE ATT&CK Framework:**  
A comprehensive knowledge base documenting real-world adversary behavior. It presented as a matrix where:
- **Tactics (Columns):** Represents the attacker's immediate goal.
- **Techniques (Cells):** Represents the specific method used to achieve a tactic.
- **Sub-Techniques:** Describes exactly how a technique is executed.

**Pyramid of Pain:**  
A conceptual model demonstrating how difficult it is for an adversary to alter their indicators when defenders block them.
|    Level    |               Indicator Type               |              Impact on Adversary if Blocked               |
|:-----------:|:------------------------------------------:|:---------------------------------------------------------:|
|   Trivial   |                Hash Values                 |   None (Simply change a single bit to change the hash)    |
|    Easy     |                IP Addresses                |         Minimal (Switch to a new proxy / VPS host)        |
|   Simple    |                Domain Names                |            Low (Register a new lookalike domain)          |
|  Annoying   |           Network/Host Artifacts           |   Moderate (Alter URI strings or malware registry keys)   |
| Challenging |                   Tools                    |     High (Rewrite/compile a completely new software)      |
|    Tough    | TTPs (Tactics, Techniques, and Procedures) | Devastating (Invent a fundamentally new way of operating) |

<br>

#### **Section 3:** Incident Handling Process Overview
<img alt="Incident Handling Process" src="./Images/D0001/Image-1.png" align="right" width="400">
The incident handling lifecycle is non-linear and highly cyclic. As evidence is discovered, the investigation scope and recovery steps dynamically evolve.

**Two Core Pillars of IR:**
1. **Investigation:** Aims to uncover the initial victim (Patient Zero), map a comprehensive incident timeline, identify the adversary's toolkit/malware, and document all compromised assets.
2. **Recovery:** Engineering and executing a precise recovery strategy to restore disrupted business operations to a verified normal state.

<br>

#### **Section 4:** Phase 1 - Preparation
Preparation is split into two primary objectives: establishing core incident response capabilities and proactively hardening the organization's defense posture to prevent incidents.

**1. Operations, Documentation, and Policy:**  
A prepared SOC depends on up-to-date documentation divided into four operational quadrants:
- **Contact Lists:** Internal escalation nodes (IR team, Legal, PR, IT) and external entities (ISPs, Law Enforcement, regulatory bodies).
- **IR Frameworks:** Standard Operating Procedures (SOPs), information-sharing protocols, and forensic investigation cheat sheets.
- **Technical Baselines:** Updated network topology diagrams, Asset Management Databases (CMDBs), and clean-state golden image configurations.
- **Emergency Powers:** Pre-authorized rapid budgets for emergency tool procurement and immediate, high-privilege emergency admin accounts.

**Real-Time Incident Logging:** During active, high-stress incidents, continuous note-taking is mandatory. Analysts must record the Who, What, When, Where, Why, and How for every defensive action. Every log entry requires a precise timestamp, the activity details, the outcome, and the operator's name. This ensures a clean judicial chain-of-custody and accurate post-incident reconstruction.

**2. Hardware and Software Toolkits**  
Incident response teams must maintain pre-configured, accessible software and hardware infrastructure:
- **SOC Jump Bag:** A deployment-ready physical kit containing:
    - **Forensics and Integrity:** Dedicated clean hard drives and hardware write-blockers to preserve drive integrity during imaging.
    - **Network & Hardware:** Spare network switches, ethernet patches, power cables, and physical disassembly teardown tools (screwdrivers, tweezers).
    - **Physical Security:** Access to a physically locked, highly secure facility to store evidence.
- **DFIR Software Architecture:**
    - **Isolated Workstations:** Air-gapped forensic laptops where host security systems can be disabled to analyze live malware without network cross-contamination.
    - **Analysis Suites:** Dedicated memory capture tools, packet analyzers, and live-response triage collectors capable of sweeping enterprise endpoints for Indicators of Compromise (IOCs).
- **Out-of-Band (OOB) Infrastructure:** Analysts must operate under the assumption that the corporate network is fully compromised and monitored by the adversary. All incident management tracking, documentation, and coordination must run on completely isolated external channels independent of corporate email or Active Directory infrastructure.

**3. Proactive Technical Hardening:**  
**Email Security (DMARC):**  
DMARC builds upon SPF and DKIM to drop spoofed corporate emails. Implementing granular email filtering rules based on DMARC header evaluation allows the SOC to actively spot and analyze external phishing campaigns masquerading as trusted internal or third-party domains.

**Endpoint Hardening and EDR:**  
Endpoints are the primary entry point for attacks. Hardening must be applied via trusted industry baselines (CIS / Microsoft benchmarks):
- **Credential Hygiene:** Disable legacy, vulnerable discovery protocols like LLMNR and NetBIOS. Implement Local Administrator Password Solution (LAPS) to rotate unique local admin credentials and strip admin permissions from standard users.
- **Execution Restrictions:** Constrain PowerShell access and explicitly block untrusted binaries from executing out of user-writable directories like \Downloads\ or \Desktop\\.
- **Scripting Controls:** Block high-risk administrative extensions (.vbs, .bat, .js) and continuously monitor LOLBins (Living off the Land Binaries) that leverage trusted native OS tools to bypass defenses.
- **Network Isolation:** Enforce host-based endpoint firewalls to block workstation-to-workstation communication, cutting off an attacker's ability to move laterally.
- **Advanced Telemetry:** Deploy modern Endpoint Detection and Response (EDR) utilities tied directly into the Anti-Malware Scan Interface (AMSI) to intercept and analyze heavily obfuscated scripts in memory before execution.

**Network Architecture and Identity:**  
- **Segmentation:** Group critical systems into separate network zones and enforce strict DMZ boundaries for public-facing assets.
- **IDS/IPS:** Perform inline SSL/TLS interception to analyze actual traffic payload data instead of relying on weak IP reputation lists.
- **Access Control:** Use 802.1X for on-premise hardware validation to drop rogue BYOD implants, and enforce cloud-based Conditional Access Policies to require company-managed compliance states before allowing authentication.
- **Privileged Identity Management:** Enforce long, robust passphrases like "i LIK3 my coffeE warm" rather than short, complex, predictable formulas like "Password1!". Mandate Multi-Factor Authentication (MFA) across all administrative access layers.

**Assessments and Readiness:**  
- **Vulnerability Management:** Run automated, continuous enterprise vulnerability scans, prioritizing manual remediation for all Critical and High findings. If patching is impossible due to legacy dependencies, isolate the vulnerable assets completely.
- **Active Directory Assessments:** Perform regular security audits of Active Directory to uncover nested privileges, misconfigured ACLs, and one-step privilege escalation paths. Never assume sysadmins are caught up on every new AD vulnerability.
- **User Awareness Training:** Deploy recurring, unannounced security testing exercises (mock phishing campaigns, physical USB drops) to turn employees into effective human detection nodes.

<br>

#### **Section 5:** Phase 2 - Detection and Analysis
Defenders can flag threat indicators using many different vectors: security alert logs (SIEM, EDR, IDS, Firewalls), proactive internal threat hunting, employee reports, or external notifications (CISA or threat intel partners).

**1. Multi-Tiered Detection Strategy:**  
To maximize visibility, design detection layers across the entire infrastructure footprint:
- **Perimeter:** External firewalls, perimeter IDS/IPS, and public DMZ logging.
- **Internal Network:** Internal routing boundaries, VLAN controls, and host-based firewalls.
- **Endpoint:** Host AV engines, EDR hooks, and memory inspection alerts.
- **Application:** Centralized application, database, and system service logs.

**2. Triage, Context, and the Timeline**  
When an alert triggers, investigators must resist the urge to immediately signal an emergency alert. Establish context first. A high-privileged connection to a foreign IP is contextless without identifying the business purpose of the asset or the user's location. Establishing context prevents costly false positives and determines whether you are dealing with a compromised executive laptop or a low-impact testing node.

During initial triage, package data into four vital quadrants:
- **Incident Metadata:** Date/time of detection, reporting vector, and specific attack classification.
- **Asset Specifications:** Comprehensive host profiles (hostname, IP/MAC, physical location, OS build, business owner, data sensitivity level).
- **Activity Status:** Active audit of who is authenticated, what actions are running, and whether the malicious payload is actively executing or stopped.
- **Forensic Evidence:** Specific malware names, cryptographic hashes (MD5/SHA256), local file drops, and associated network indicators.

**Developing the Incident Timeline:**  
Compile collected artifacts chronologically into a standardized incident timeline. Because forensic evidence is gathered out of order, the timeline acts as the single source of truth for tracking the attack.
|    Date    | Time (With Zone) |  Hostname   |               Event Description              |  Data Source  |
|:----------:|:----------------:|:-----------:|:--------------------------------------------:|:-------------:|
| 05/29/2026 |    13:31 EEST    | SQL-PROD-01 | Adversary tool "Mimikatz" detected in memory | EDR/AMSI Hook |

**3. Technical Investigation and the IOC Cycle:**  
<img alt="Incident Handling Process" src="./Images/D0001/Image-2.png" align="right" width="400">
Never completely wipe and rebuild a compromised server without performing technical root-cause analysis. Doing so deletes critical evidence and leaves the original vulnerability unpatched, allowing the adversary to easily compromise the newly rebuilt server using the exact same exploit path.

The forensic technical investigation runs as a highly cyclic three-step process:
1. **Creation and Deployment of IOCs:** Standardize threat artifacts (IP addresses, domains, file names, file hashes) into structured formats like YARA or OpenIOC using creation suites (such as Mandiant IOC Editor). Machine-readable intelligence feeds are distributed across entities using standardized JSON-based STIX formats.
2. **Identification of New Leads:** Sweep the enterprise environment using your newly engineered IOC signatures. Filter and prioritize hits to eliminate generic false positives and zero in on authentic, newly compromised systems.
3. **Data Collection and Forensic Analysis:** Perform live response data collection from the flagged endpoints without disrupting operations or modifying core memory states.

**Critical Credential Hygiene:**  
During live-response investigation, analysts must ensure they do not introduce credential leakage. Never use administrative utilities that cache high-privileged credentials onto an untrusted, compromised host.
- **Secure Methods:** Use WinRM or Windows Logon Type 3 (Network Logon) since these execute actions safely without caching credentials locally.
- **Tool Nuance:** Tools like PsExec will securely run actions without caching credentials when using the current user's active token session, but they will cache high-privileged credentials if they are explicitly typed out in plain text in the command line.

**4. Modernizing Detection with Artificial Intelligence:**  
Modern operations supplement traditional manual alert triage with integrated AI modules. AI models optimize incident workflows by replacing slow manual log parsing with real-time, automated analysis. By processing historical incident data, AI platforms detect anomalous behavioral patterns and complex threat techniques significantly faster than standard human parsing cycles.

<br>

#### **Section 6:** Containment, Eradication, and Recovery
Once an incident is verified, handlers must systematically execute the containment, eradication, and recovery strategy.
**1. Containment Strategy:**  
Containment stops an attack from spreading across the network and prevents data exfiltration. Coordinated execution is essential: if multiple systems are compromised, containment actions must be executed across all affected nodes simultaneously. Isolating systems one-by-one tips off the attacker, allowing them to shift techniques, jump networks, or immediately execute ransomware payloads.
- **Short-Term Containment:** Immediate, minimal-footprint isolation measures. This includes placing endpoints into isolated containment VLANs, pulling physical network cables, and redirecting malicious C2 traffic via DNS sinkholing, all while ensuring active memory and local storage are preserved for forensics.
- **Long-Term Containment:** Strategic, permanent system modifications. This includes executing enterprise-wide password resets, updating firewall rulesets, applying immediate security patches to edge routing nodes, and terminating rogue processes after coordinating with business stakeholders.

**2. Eradication Phase:**  
Eradication ensures that the threat is completely removed and the adversary is evicted from the internal environment.
- **Threat Removal:** Find and delete all secondary implants, persistent scripts, rogue scheduled tasks, and residual attacker files.
- **System Restoration:** Rebuild compromised infrastructure completely from verified bare-metal golden images, or securely restore systems using known-clean historical backups.
- **Infrastructure Hardening:** Apply proactive configurations and software security updates across the entire enterprise directory footprint to guarantee the adversary cannot reuse the same access vector.

**3. Recovery Phase:**  
Recovery returns verified, hardened assets back to production while validating system integrity.
- **Production Return:** Re-introduce systems to the live environment in close collaboration with business managers, ensuring functionality is normal and business processes are stable.
- **Aggressive Monitoring:** Place recovered networks under intense monitoring loops, checking for unusual logins, unauthorized registry edits, or rogue child processes.
- **Phased Security Implementation:** Fix easily exploitable vulnerabilities immediately before rolling out long-term, systemic infrastructure upgrades over the following weeks and months.

<br>

#### **Section 7:** Post-Incident Activity (Lessons Learned)
The incident handling cycle is complete only after the team documents the lifecycle of the breach and runs a retrospective review to optimize future security controls.
**1. Core Objectives:**  
- **Objective Documentation:** Finalize an authoritative, clear incident report to preserve a clean record of the attack.
- **Capability Improvement:** Extract operational lessons learned to improve playbook workflows, adjust alert thresholds, and patch defensive gaps.
- **Stakeholder Debriefing:** Convene a mandatory debriefing meeting with all involved internal and external parties (IT, Legal, PR, Management) within days of report completion to review team response metrics and performance.

**2. Reporting Metrics and Operational Value:**  
A comprehensive, professional incident report must detail:
- The exact step-by-step incident timeline.
- Team playbook compliance and response efficiency.
- Eradication steps and preventive actions taken.
- **Core Performance Metrics:** Total incident volume handled, Mean Time to Detect (MTTD), Mean Time to Remediate (MTTR), and total financial/operational damage.

The final report provides essential documentation for legal defense and court proceedings, serves as a high-value training scenario for onboarding new SOC team analysts, and acts as the justification required to update enterprise corporate security policies, modify team structures, and purchase advanced security tools.