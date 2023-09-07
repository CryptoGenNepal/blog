---
title: "Analysis on malware imposing as adult content of Nepali celebrity"
date: 2023-09-07T00:00:00+05:45
# post image
image: "/images/blog/analysis-on-malware-imposing-as-adult-content-of-nepali-celebrity/cover.png"
# author
author: "Venus Chhantel"
# post type (regular/featured)
type: "regular"
# meta description
description: "This year, in July, a new “MalDoc in PDF” attack which could evade detection and analysis was shared by JPCERT. This malware was polyglot, meaning a file that combine two or more file formats in a way that it could be executed as more than one file type by different application without error to evade detection and hinder analysis tools. In this case, the malware sample could be opened either as Word or as PDF and had embedded MHT (MIME HTML) that encoded macro in its ActiveMime."
# post draft
draft: false
---

{{< toc >}}

## Introduction:
This year, in July, a new “MalDoc in PDF” attack which could evade detection and analysis was shared by JPCERT. This malware was polyglot, meaning a file that combine two or more file formats in a way that it could be executed as more than one file type by different application without error to evade detection and hinder analysis tools. In this case, the malware sample could be opened either as Word or as PDF and had embedded MHT (MIME HTML) that encoded macro in its ActiveMime.

## Identification:
**Source:** [Triage](https://tria.ge/230829-clz17ahd83)
**SHA256:** ef59d7038cfd565fd65bae12588810d5361df938244ebad33b71882dcf683058
**AV-Detection:** 22/59 ****([VirusTotal](https://www.virustotal.com/gui/file/ef59d7038cfd565fd65bae12588810d5361df938244ebad33b71882dcf683058))


## Analysis:
### Static Analysis:
The sample collected was a PDF file, which are normally initial stager. Let's start with static analysis of this sample by loading this file in VSCode. Initially, its contents looked legitimate with PDF header and objects as shown below.


But scrolling through the contents, there is MIME version declaration as shown below. From this part, the MHT (MIME HTML) content starts. Since this sample is polyglot and can be executed as Word file, the Word file can render this MHT contents.


