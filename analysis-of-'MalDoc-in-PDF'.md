---
title: "Analysis on “MalDoc in PDF”"
date: 2023-09-07T00:00:00+05:45
# post image
image: "https://github.com/CryptoGenNepal/blog/assets/142308575/5a41bafd-9bcd-4286-b4de-f7014efec031"
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

**AV-Detection:** 22/59 ([VirusTotal](https://www.virustotal.com/gui/file/ef59d7038cfd565fd65bae12588810d5361df938244ebad33b71882dcf683058))

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/4a68ee39-057c-4933-b148-9f0b89b5d6e4" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

## Analysis
### Static Analysis
The sample collected was a PDF file, which are normally initial stager. Let's start with static analysis of this sample by loading this file in VSCode. Initially, its contents looked legitimate with PDF header and objects as shown below.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/581f1b0c-050b-4a75-acc3-890b794dfd4a" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

But scrolling through the contents, there is MIME version declaration as shown below. From this part, the MHT (MIME HTML) content starts. Since this sample is polyglot and can be executed as Word file, the Word file can render this MHT contents.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/4e8c4853-382c-4c9e-b786-753879dcfa4d" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

The <link rel=’Edit-Time-Date’> object inside the MHT content point to ActiveMime. Here, ActiveMime is an undocumented Microsoft file format that is often used to encode the macro.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/fe25074e-a2d3-4c8b-832d-52f9a6d6458e" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Also in above figure, note that the referenced location (href) to the ActiveMime of ‘Edit-Time-Date’ object is encoded, which was decoded using CyberChef as:

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/6ad1d04a-8104-4c82-bb56-7fd4ba881418" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Using decoded location path name ‘lonhzFH_files/image7891805.jpg’, it was searched in VScode and jumped to that part as shown below. The content (ActiveMime) is obfuscated text with lots of spaces.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/b78c26c0-9aec-4e0d-abca-df690a3f96b4" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/64982a1a-a898-4a08-b0be-08efdf12ffc4" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

The contents from above were copied to CyberChef where it revealed that those spaces were actually created by CR (Carriage Return)’.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/2ac3e808-b251-4d03-9c09-cac4bcd2cdc9" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

The content was then decoded from Base64 revealing it to be ActiveMime, which encode the macro. So, it was then saved as macro.bin.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/cd762228-6811-4bf1-bd76-55d9f417f144" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

To search for the embedded macro inside the decoded content, binwalk was used.

    binwalk macro.bin

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/006dda93-d5ba-49ea-b5d6-db2a6fe388ce" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

The binwalk revealed a compressed data, which is part of the ActiveMime format that encode the macro. It was again extracted using binwalk.

    binwalk -D='.*' macro.bin

After extracting, it revealed a file named 32, as shown below.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/b1fcea13-f2d7-41b0-859e-cdcabb34cb35" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

The above extracted file should be the decoded malicious macro. So, it was checked using the Oletools. First, Oleid was used to check for any macro.

    oleid 32

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/96236ddb-0ed7-44d1-b64a-e4853aa43e6b" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

The Oleid tool found VBA macros within it.

To extract the VBA macro inside it, Olevba tool was used.

    olevba 32

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/30d18dc4-fc0c-473f-a9a5-48461db07bee" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Here, from the output of olevba tool, it can be found that the macro will download a msi file from its C2 server (hxxps[:]//cloudmetricsapp[.]com/wp-content/uploads/docs/addin[.]msi). Then, it will execute the downloaded msi file with Office.InstallProduct. Also, this macro will be executed as soon as the Word file is opened through AutoExec, and download and install that msi file.

This further verify that this pdf sample file is an initial stager that will bypass detection and download another payload, which is the msi file.

Using the URL obtained from the macro, it was checked in Browserling sandbox if it acts as downloader. But it was unreachable as shown below.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/3231b5ec-f689-49a4-be03-2c5be4b084f2" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

### Dynamic Analysis:
Let's now further analyze this sample through dynamic analysis. The PDF file was saved as Word file and opened. Notice the ‘Enable Content’ is still there. Although this sample was able to bypass detection, it was not able to bypass this protection measure.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/e95a9caf-a52f-4b44-9718-1747d838a717" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

When the ‘Enable Content’ is enabled, the Word will render the MHT contents, which will decode the macro from the ActiveMime blob and execute it to download the msi file as discovered during static analysis.

In the Wireshark capture, it can be seen that it is resolving the domain cloudmetricsapp[.]com to IP address of 179[.]60[.]147[.]105. After that the infected host tried to initiate connection over 443 to download the msi file but get no reply from server as shown below.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/3b309d87-e984-4ccb-ac87-28ac7222402e" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

It is clear now that this ‘MalDoc in PDF’ is an initial stager that will download its final payload by bypassing detection. And there have been many scenarios where one malware family has been used as initial stager for other malware families. So, in near future, this malware could probably be used by other malware families to act as their initial stager. So, the next section will include YARA rule for detecting these kind of malware samples.

## Detection with YARA:

    rule MalDocinPDF{
    meta:
        description= "Detecting MalDocs in PDF"

    strings:
        $mht0 = "mime" ascii nocase
        $mht1 = "content-location:" ascii nocase
        $mht2 = "content-type:" ascii nocase
        $mht3 = "Edit-Time-Data" ascii nocase
        $doc = "<w:WordDocument>" ascii nocase
        $xls = "<x:ExcelWorkbook>" ascii nocase
     condition:
        (uint32(0) == 0x46445025) and
        (2 of ($mht*)) and 
        ( (1 of ($doc)) or 
          (1 of ($xls)) )
      }

## IoC:
Explore IoC of ‘MalDoc in PDF’ from Virus Total Graph as they have great visualization.

Link: ([VirusTotal Graph](https://www.virustotal.com/graph/gf02630da298546129218c8f0577ea4a751c3137cfcb54c56b785a69b33d5372f))

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/394122b8-ec2f-413d-9588-07e746669690" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}
