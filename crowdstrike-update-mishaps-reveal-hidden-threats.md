---
title: "Hunting Down Remcos RAT: How CrowdStrike Update Mishaps Can Reveal Hidden Threats"
date: 2024-07-22T12:00:00+05:45
# post image
image: "images/blog/crowdstrike-update-mishaps-reveal-hidden-threats/crowdsttike-remcos.png"
# author
author: "Simran Karki"
# post type (regular/featured)
type: "featured"
# meta description
description: "The cybersecurity community faced a significant challenge worldwide due to a BSOD (Blue Screen of Death) error caused by a CrowdStrike update. As attackers attempt to exploit this situation, it is crucial for organizations to strengthen their defenses and actively hunt for potential threats. Here’s a comprehensive guide on how to conduct threat hunting for this incident."
# post draft
draft: false
---

{{< toc >}}

## Background

Recently, the cybersecurity community faced a significant challenge worldwide due to a BSOD (Blue Screen of Death) error caused by a CrowdStrike update. As attackers attempt to exploit this situation, it is crucial for organizations to strengthen their defenses and actively hunt for potential threats. Here’s a comprehensive guide on how to conduct threat hunting for this incident.

## Threat Landscape

Threat actors are now actively exploiting this incident to target CrowdStrike customers through various malicious activities. The original issue stemmed from a content update for the CrowdStrike Falcon sensor on Windows hosts, which caused system crashes and blue screens on affected machines.

CrowdStrike Intelligence has reported several tactics employed by these malicious actors:

**Phishing Emails:** Cybercriminals are sending fraudulent emails posing as CrowdStrike support, attempting to trick customers into revealing sensitive information or granting unauthorized access.
Impersonation on Phone Calls: There have been instances of threat actors impersonating CrowdStrike staff during phone calls, likely aiming to manipulate victims into compromising their security.
**Fake Researchers:** Some attackers are presenting themselves as independent researchers, falsely claiming to have evidence linking the technical issue to a cyberattack and offering dubious remediation advice.
**Malicious Scripts:** Criminals are attempting to sell scripts that supposedly automate recovery from the content update issue, which may instead introduce malware or create new vulnerabilities.

## Threat Hunting

To address this evolving threat landscape, the threat hunting process outlines a systematic approach to detect, investigate, and mitigate these malicious activities, ensuring the security and integrity of affected systems and users.

Threat hunting can be conducted through various methods, as I discussed in previous blogs.

[Link to my previous blogs on threat hunting](https://cryptogennepal.com/blog/threat-hunting-with-windows-event-logs/)

1. IOC Based Hunting
2. Hypothesis Based Hunting
3. Baseline Based Hunting
4. Anomaly Based Hunting

In this blog, I will focus on hypothesis-based threat hunting, using the hypothesis outlined below.

## Importance of Hypothesis in Threat Hunting

Formulating a hypothesis is crucial in threat hunting because a well-defined hypothesis directs the investigation, focusing on specific threats or activities. Hypotheses provide a structured approach to threat hunting, ensuring that the process follows a logical sequence. This structure helps in systematically identifying, analyzing, and mitigating threats.

**Hypothesis: Cybercriminals Exploit CrowdStrike Update Mishap to Distribute Remcos RAT Malware**

## Methodologies of Hypothesis Based hunting

P.S. This is my own methodology for threat hunting. If you have any suggestions or modifications, feel free to share or modify.

### Plan

1. Formulate your hypothesis as demonstrated above.
2. Define the goal of your hunt. For this hunt, my goal is to determine if Remcos exists in my network. The primary objective is to validate or invalidate my hypothesis.
3. Research the technical context, including Indicators of Compromise (IOCs), attack behaviors/patterns, and similar incidents.
4. Build analytics for the hunt if you are using SIEM or EDR tools.

```SQL
For example: (source_address or destination_address in IOC and action=”allow” | chart count() by source_address, destination_address,action,device_name )
```

### **Document**

Create a plan document that includes your hypothesis, hunting goals, the analytics you will use, and the technical context.

### IOCs Gathering

To begin, gather all information related to the attacks happening recently due to the CrowdStrike incident, such as IOCs (Indicators of Compromise) and TTPs (Tactics, Techniques, and Procedures). Some IOCs that have been identified include:

**Phishing Email Domains**:

- crowdstrike.phpartners[.]org
- crowdstrike0day[.]com
- crowdstrikebluescreen[.]com
- crowdstrike-bsod[.]com
- crowdstrikeupdate[.]com
- crowdstrikebsod[.]com
- www.crowdstrike0day[.]com
- www.fix-crowdstrike-bsod[.]com
- crowdstrikeoutage[.]info
- www.microsoftcrowdstrike[.]com
- crowdstrikeodayl[.]com
- crowdstrike[.]buzz
- www.crowdstriketoken[.]com
- www.crowdstrikefix[.]com
- fix-crowdstrike-apocalypse[.]com
- microsoftcrowdstrike[.]com
- crowdstrikedoomsday[.]com
- crowdstrikedown[.]com
- whatiscrowdstrike[.]com
- crowdstrike-helpdesk[.]com
- crowdstrikefix[.]com
- fix-crowdstrike-bsod[.]com
- crowdstrikedown[.]site
- crowdstuck[.]org
- crowdfalcon-immed-update[.]com
- crowdstriketoken[.]com
- crowdstrikeclaim[.]com
- crowdstrikeblueteam[.]com
- crowdstrikefix[.]zip
- crowdstrikereport[.]com

**Domains Impersonating CrowdStrike**:

- crowdstrike.phpartners[.]org
- crowdstrike0day[.]com
- crowdstrikebluescreen[.]com
- crowdstrike-bsod[.]com
- crowdstrikeupdate[.]com
- crowdstrikebsod[.]com
- www.crowdstrike0day[.]com
- www.fix-crowdstrike-bsod[.]com
- crowdstrikeoutage[.]info
- www.microsoftcrowdstrike[.]com
- crowdstrikeodayl[.]com
- crowdstrike[.]buzz
- www.crowdstriketoken[.]com
- www.crowdstrikefix[.]com
- fix-crowdstrike-apocalypse[.]com
- microsoftcrowdstrike[.]com
- crowdstrikedoomsday[.]com
- crowdstrikedown[.]com
- whatiscrowdstrike[.]com
- crowdstrike-helpdesk[.]com
- crowdstrikefix[.]com
- fix-crowdstrike-bsod[.]com
- crowdstrikedown[.]site
- crowdstuck[.]org
- crowdfalcon-immed-update[.]com
- crowdstriketoken[.]com
- crowdstrikeclaim[.]com
- crowdstrikeblueteam[.]com
- crowdstrikefix[.]zip
- crowdstrikereport[.]com

**Malicious File Names and Hashes**:

- **crowdstrike-hotfix.zip**: c44506fe6e1ede5a104008755abf5b6ace51f1a84ad656a2dccc7f2c39c0eca2
- **Setup.exe**: 5ae3838d77c2102766538f783d0a4b4205e7d2cdba4e0ad2ab332dc8ab32fea9
- **madBasic\_.bpl**: d6d5ff8e9dc6d2b195a6715280c2f1ba471048a7ce68d256040672b801fda0ea
- **maidenhair.cfg**: 931308cfe733376e19d6cd2401e27f8b2945cec0b9c696aebe7029ea76d45bf6
- **RemCos Payload**: 48a3398bbbf24ecd64c27cb2a31e69a6b60e9a69f33fe191bcf5fddbabd9e184

**RemCos C2 Address**:

- 213.5.130[.]58[:]443

### TTPs Information

Research on the TTPs that aligns with your hypothesis, here I will be focusing on the Remcos since, my hypothesis is related with Remcos RAT malware.

Mapping with the MITRE ATT&CK framework will significantly enhance threat hunting for malware by providing a structured approach to identifying and understanding adversary behavior. So, here is the information of Remcos RAT malware TTPS as per MITRE framework:

### MITRE Mapping

{{< image src="/images/blog/crowdstrike-update-mishaps-reveal-hidden-threats/RemcosMitre.png" caption="RemCos" alt="RemCos" height="" width="" position="center" command="fit" option="" class="img-fluid" title="RemCos" >}}

### Behavioral Analysis

Next, analyze the collected information, such as IOCs and TTPs, to validate your formulated hypothesis. Use the above information and the MITRE ATT&CK framework to identify techniques associated with Remcos RAT. Here are some recommended actions:

- Monitor the execution of command-line interpreters and scripting engines, such as `cmd.exe`, PowerShell, and scripting files (e.g., `.bat`, `.ps1`, `.js`).
- Look for modifications to autostart registry keys, the creation of new services, or scheduled tasks that are set to run at boot or logon.
- Identify attempts to bypass User Account Control (UAC) and other elevation control mechanisms, such as the use of `runas` or `schtasks` with elevated privileges.
- Monitor changes to the registry that could disable security features or hide the malware.
- Look for scanning activities on the file system aimed at discovering sensitive files or directories.
- Identify attempts to inject code into running processes, which can be used to remain hidden and access process memory.
- Monitor for the use of proxies to route Command and Control (C2) traffic, which can hide the source and destination.

## Investigate

Next, conduct an in-depth investigation into the suspicious activities identified during the analysis phase. Correlate your findings across multiple data sources—such as endpoints, networks, and security devices—to verify the presence of threats.

## Mitigate

Finally, to neutralize the detected threats, inform your Incident Response (IR) team so they can take appropriate action. Ensure you provide them with all relevant details from your investigation to facilitate a swift and effective response.

**Note: This step is optional and should be followed only if your hypothesis is confirmed and there are threats that need to be mitigated.**

## Conclusion

In conclusion, assess the success of your hunt using the metrics and KPIs defined in your planning document. Document your findings, actions taken, and lessons learned from the hunt. This will help refine your methodology for future hunts and improve overall threat detection and response strategies.
