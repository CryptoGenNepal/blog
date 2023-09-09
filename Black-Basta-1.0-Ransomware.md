---
title: "Black Basta 1.0 Ransomware"
date: 2023-09-07T00:00:00+05:45
# post image
image: "https://github.com/CryptoGenNepal/blog/assets/142308575/8306881c-48e5-4af2-bd96-34824e02bc02"
# author
author: "Venus Chhantel"
# post type (regular/featured)
type: "regular"
# meta description
description: "Black Basta is a ransomware group that emerged in early 2022, targeting various organizations across multiple industries. It is believed to be a Russian group that evolved from the defunct Conti threat actor group. It operates as a ransomware-as-a-service (RaaS), offering its malware to other cybercriminals for a share of the profits. Black Basta 1.0 is the early ransomware of this group. Later the ransomware was made sophisticated leading to Black Basta 2.0 ransomware."
# post draft
draft: false
---

{{< toc >}}

This blog will be covering code level analysis of Black Basta 1.0 ransomware to unravel all the possible capabilities of this ransomware. And based on the analysis, the TTP of this ransomware will be broken down by referencing MITRE ATT&CK and YARA rules will be created for its detection.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/a82a8926-4b4c-4e04-9957-f96cb0cd7ce4" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Why analysis of Black Basta 1.0, when it’s updated Black Basta 2.0 is available? It’s because it will help to better understand Black Basta 2.0 later regarding the update trend of this ransomware.

## Introduction:
Black Basta is a ransomware group that emerged in early 2022, targeting various organizations across multiple industries. It is believed to be a Russian group that evolved from the defunct Conti threat actor group. It operates as a ransomware-as-a-service (RaaS), offering its malware to other cybercriminals for a share of the profits. Black Basta 1.0 is the early ransomware of this group. Later the ransomware was made sophisticated leading to Black Basta 2.0 ransomware.

## Case Study: Black Basta 1.0
### Identification:
Source: ([Malware Bazaar](https://bazaar.abuse.ch/sample/5d2204f3a20e163120f52a2e3595db19890050b2faa96c6cba6b094b0a52b0aa/))

SHA256: 5d2204f3a20e163120f52a2e3595db19890050b2faa96c6cba6b094b0a52b0aa

AV Detection: 60/70 detected ([VirusTotal](https://www.virustotal.com/gui/file/5d2204f3a20e163120f52a2e3595db19890050b2faa96c6cba6b094b0a52b0aa))

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/82616e87-b7a3-4bdc-80f8-4e356d4394f4" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

### Analysis:
The sample collected from Malware Bazaar was loaded in the x64dbg for code level analysis through debugging.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/a17f6327-acdd-4f87-ac00-467061c1e9e0" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

In the above figure, at offset 0x001CC5CA, it can be seen that “ENCRYPTION” string is pushed onto the stack and at offset 001CC5D4, there is call to a function that make it display on the sample’s CLI. After displaying this message in its CLI, the sample starts its malicious activities.

After that, there is call to OpenSCManager at offset 001CC642 as shown below.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/4e75b2a3-2949-4b88-ad02-a8a7747d8a31" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Before the call to the function, there are three parameters passed with push instructions, which are:

-   lpMachineName [esp] = 0x0, mean local machine
-   lpDatabaseName [esp+4] = 0x0, mean the SERVICES_ACTIVE_DATABASE
-   dwDesiredAccess [esp+8] = 0xF003F, mean the STANDARD_RIGHTS_REQUIRED

When this function executes successfully, it returns a handle to specified service control manager database, else fail then returns NULL.

-   In this case, the function was executed successfully so the return value (stored in eax) is a handle.
-   After the call to the function, there is  **mov esi, eax**  instruction which will store the value of eax in esi.
-   Then, there is another  **test esi, esi**  instruction, which will perform bitwise AND operation on esi with itself. And since the value is a handle (non-zero value), the result of AND operation of 1 and 1 is 1. So, ZF = 0.
-   In the next instruction there is  **jne**, and since ZF was not set, it will take the jump and move to offset 0x001CC69E.

With this the sample has established a successful connection to the service control manager with which it can create, delete and modify any services.

If the function call had failed, return value would be NULL (zero) so will not take the jump and GetLastError function will be called and display either “Please run program as admin” or “Cant open scm manager: ” on its CLI as per return code. This showed us that the sample require administrator level of privilege.

Moving on, after the above jump with  **jne**, there was call to a function at offset 0x001CC69E. When step into this function, there is interesting things occurring as shown below.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/7f8aab03-b58d-47b8-9747-a483c19e5abc" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

A command that is using vss(Volume Shadow Copy Service) utility to delete all the shadow copies silently is being pushed onto the stack and then a function is called to execute the command. With this, the sample wants to prevent use of shadow copies for possible file recovery which will be later encrypted.

    C:\\Windows\\SysNative\\vssadmin.exe delete shadows /all /quiet
    C:\\Windows\\System32\\vssadmin.exe delete shadows /all /quiet

This can also further be verified from the Process Monitor capture of this sample as shown below, where there is a process created of cmd.exe which is executing the above-mentioned command.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/b21db105-4f29-4fcb-a2be-712ef428c38f" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

After this, the sample push “C:\Users\husky\AppData\Local\Temp\dlaksjdoiwq.jpg” to the stack and with a function call, it saves the image dlaksjdoiwq.jpg under the C:\Users\husky\AppData\Local\Temp directory.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/e9fdd370-bfaf-4762-81bf-dfc51d397f6e" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

This can also be verified from Process Monitor capture.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/3b3aeb3b-2ac1-45ca-a671-79bcbf4c7fa4" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

The image file dropped can be seen in the C:\Users\husky\AppData\Local\Temp directory.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/a5a2ae03-80dc-4e38-a307-031ba1917c5a" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

After that there is call to SystemParameterInfo API and its parameter are passed before the call using push instruction.

-   uiAction [esp] = 0x14, mean SPI_SETDESKWALLPAPER
-   uiParam [esp+4] = 0x0
-   pvParam [esp + 8] = 0xD2D740 L”C:\\Users\\husky\\AppData\\Local\\Temp\\dlaksjdoiwq.jpg”

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/c4e01fa6-4309-4fc5-b10f-f79b10e408c2" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

With this API call with those passed parameters, it sets up the previously dropped image file dlaksjdoiwq.jpg as wallpaper as shown below.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/82ec69e3-127f-4f95-88da-e7ff42025fd9" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

This can also be verified from the Process Monitor capture where Wallpaper registry value is set under HKCU\Control Panel\Desktop.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/3da340b8-8635-4f79-9ee7-97f163c6fbea" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Checking the registry under Registry Editor, the set value can be seen, which is the previously dropped image file.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/59d97612-cd70-4f4f-afcc-d5faa56bc4dd" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

After this, C:\Users\husky\AppData\Local\Temp\fkdjsadasd.ico is pushed onto the stack and after a function call, the fkdjsadasd.ico icon file is saved under C:\Users\husky\AppData\Local\Temp. This icon file will be later used by the sample to set up the icon for encrypted files.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/6e72acf4-67cb-4978-94a9-3c764bbf214c" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

This can also be verified from Process Monitor capture.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/61db4680-6f25-4746-b70c-336b4ab76beb" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

The icon dropped by the sample is shown below.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/54e779e8-02e6-40b7-a774-b3ce3be97f1a" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

After this, the sample calls GetServiceDisplayName API to check for service. In this case, the sample is checking for Fax service.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/1c5b4dde-fa41-4b4c-b956-81ddd3bbbef5" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

In this case:

-   There was Fax service in the machine, so the return value stored in eax is a handle (non-zero value).
-   After the call, there is  **test eax, eax**  instruction which performs bitwise AND operation on eax with itself. And the bitwise AND operation for 1 and 1 is 1. So, ZF = 0.
-   So, in next  **je**  instruction, it does not take the jump as ZF is not set.

Without taking the jump, the program then moves to another API call, which is DeleteService.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/7ca7c973-9deb-42a1-a262-f50b5592fe98" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

With this API call, it deletes the default Fax service.

After deleting the Fax service, it then calls CreateService API to create a new service.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/bb09e5e8-56b7-4e0c-859e-b2650fcea5fe" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

The parameter passed to the CreateService using push instruction can be seen before the call to this API. Some interesting parameters pushed are:

-   lpServiceName =  **push eax**  = Fax
-   lpDisplayName =  **push ecx**  = Fax
-   dwServiceType =  **push 10**  = SERVICE_WIN32_OWN_PROCESS (Service starts as its own process)
-   dwStartType =  **push 2**  = SERVICE_AUTO_START (Start automatically at system startup)
-   lpBinaryPathName =  **push edx**  = L “C:\\Users\\husky\\Desktop\\Black Basta\\Black Basta.exe”

Based on the parameters of the CreateService API call, it was found that it creates a Fax service pointing to the original Black Basta executable file that will start automatically as its own process at system startup. The new service created is Fax to disguise as legitimate Fax service (legitimate Fax service previously deleted).

After this there is call to GetSystemMetrics API to check the boot options.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/6a942fef-2578-438f-b332-053f752eb725" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Then, there is call another API, RegOpenKey, which can be seen in figure below opening the SYSTEM\CurrentControlSet\Control\SafeBoot\Network registry key.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/75bd222e-6fb6-4d9d-8cdb-4e9678ec112f" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

After that there is call to RegCreateKey API, which can be seen in figure below, is adding the Fax value under previously opened Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot\Network registry. This will make the system to start the previously disguised Fax service during safe boot mode.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/a722073c-f559-4f0d-9cfe-c91be31fa0e5" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

This opening and setting value in that registry key can also be verified from Process Monitor capture.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/99526d9a-c503-4ca7-81c8-f703f0b0eae6" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Furthermore, it was verified from Registry Editor.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/401f0abd-5d9c-4962-9de5-d7304d70c102" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

After this, some commands are pushed onto the stack and functions are called to execute those commands.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/5245af78-8f34-44a2-8af9-630d3b765a40" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

    bcdedit /set safeboot network   
    C:\\Windows\\System32\\bcdedit.exe /set safeboot network
    C:\\Windows\\SysNative\\bcdedit.exe /set safeboot network

This command will set a boot entry option to start in safe mode with networking. Also, it has already set disguised Fax service to run itself during safe mode and start its encryption.

This command execution can also be verified from Process Monitor capture.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/5c75c8c4-a512-4f6c-a890-8e58221cb0d4" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Finally, the sample calls ShellExecute API, to open command prompt to execute its command as specified by its 4th parameter (/C shutdown -r -f -t 0) causing the system to restart immediately forcing all application to close.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/30365919-8fea-4c22-9de2-c335894a8a30" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Since, the system was set to start in safe mode previously by the sample, after the restart, the system opens up in safe mode. Also, the sample had set the Fax service to execute during safe mode causing the file to be encrypted after the system startup as shown below.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/aa0c29d7-1832-45d9-b1e9-a0ae107f07a4" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Files are encrypted with .basta extension and the previously dropped fkdjsadasd.ico under C:\Users\husky\AppData\Local\Temp is used as icon for those encrypted files.

## MITRE ATT&CK TTP:

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/c97b1849-aae0-4739-8013-bdd5b1a082e9" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

## Detection with YARA:

    rule Black_Basta_Ransomware{  
    meta:  
	    author= "Venus Chhantel"  
	    filetype= "Win32 exe"  
	    description= "Detecting Black Basta 1.0 Ransomware"  
    strings:  
	    $ransom_note= {59 6F 75  72  20  64  61  74  61  20  61  72  65  20  73  74 6F 6C 65 6E 20  61 6E 64  20  65 6E 63  72  79  70  74  65  64 0D 0A 54  68  65  20  64  61  74  61  20  77  69 6C 6C 20  62  65  20  70  75  62 6C 69  73  68  65  64  20 6F 6E 20  54 4F 52  20  77  65  62  73  69  74  65  20  69  66  20  79 6F 75  20  64 6F 20 6E 6F 74  20  70  61  79  20  74  68  65  20  72  61 6E 73 6F 6D 0D 0A 59 6F 75  20  63  61 6E 20  63 6F 6E 74  61  63  74  20  75  73  20  61 6E 64  20  64  65  63  72  79  70  74  20 6F 6E 65  20  66  69 6C 65  20  66 6F 72  20  66  72  65  65  20 6F 6E 20  74  68  69  73  20  54 4F 52  20  73  69  74  65 0D 0A 28  79 6F 75  20  73  68 6F 75 6C 64  20  64 6F 77 6E 6C 6F 61  64  20  61 6E 64  20  69 6E 73  74  61 6C 6C 20  54 4F 52  20  62  72 6F 77  73  65  72  20  66  69  72  73  74  20  68  74  74  70  73 3A 2F 2F 74 6F 72  70  72 6F 6A 65  63  74 2E 6F 72  67  29}  
      
	    $string1= {45 4E 43  52  59  50  54  49 4F 4E}  
	    $string2= {3A 53  75 6E 3A 53  75 6E 64  61  79 3A 4D 6F 6E 3A 4D 6F 6E 64  61  79 3A 54  75  65 3A 54  75  65  73  64  61  79 3A 57  65  64 3A 57  65  64 6E 65  73  64  61  79 3A 54  68  75 3A 54  68  75  72  73  64  61  79 3A 46  72  69 3A 46  72  69  64  61  79 3A 53  61  74 3A 53  61  74  75  72  64  61  79}  
	    $string3= {3A 4A 61 6E 3A 4A 61 6E 75  61  72  79 3A 46  65  62 3A 46  65  62  72  75  61  72  79 3A 4D 61  72 3A 4D 61  72  63  68 3A 41  70  72 3A 41  70  72  69 6C 3A 4D 61  79 3A 4D 61  79 3A 4A 75 6E 3A 4A 75 6E 65 3A 4A 75 6C 3A 4A 75 6C 79 3A 41  75  67 3A 41  75  67  75  73  74 3A 53  65  70 3A 53  65  70  74  65 6D 62  65  72 3A 4F 63  74 3A 4F 63  74 6F 62  65  72 3A 4E 6F 76 3A 4E 6F 76  65 6D 62  65  72 3A 44  65  63 3A 44  65  63  65 6D 62  65  72}  
	    $string4= {41  42  43  44  45  46  47  48  49 4A 4B 4C 4D 4E 4F 50  51  52  53  54  55  56  57  58  59 5A 61  62  63  64  65  66  67  68  69 6A 6B 6C 6D 6E 6F 70  71  72  73  74  75  76  77  78  79 7A 30  31  32  33  34  35  36  37  38  39 2B 2F}  
      
	    $file1= {64 6C 61 6B 73 6A 64 6F 69  77  71 2E 6A 70  67}  
	    $file2= {66 6B 64 6A 73  61  64  61  73  64 2E 69  63 6F}  
	    $file3= {??  65  61  64  ??  65 2E 74  78  74}  
      
	    $cmd1= {43 3A 5C 57  69 6E 64 6F 77  73 5C 53  79  73 4E 61  74  69  76  65 5C 76  73  73  61  64 6D 69 6E 2E 65  78  65  20  64  65 6C 65  74  65  20  73  68  61  64 6F 77  73  20 2F 61 6C 6C 20 2F 71  75  69  65  74}  
	    $cmd2= {43 3A 5C 57  69 6E 64 6F 77  73 5C 53  79  73  74  65 6D 33  32 5C 76  73  73  61  64 6D 69 6E 2E 65  78  65  20  64  65 6C 65  74  65  20  73  68  61  64 6F 77  73  20 2F 61 6C 6C 20 2F 71  75  69  65  74}  
	    $cmd3= {62  63  64  65  64  69  74  20 2F 64  65 6C 65  74  65  76  61 6C 75  65  20  73  61  66  65  62 6F 6F 74}  
	    $cmd4= {43 3A 5C 57  69 6E 64 6F 77  73 5C 53  79  73  74  65 6D 33  32 5C 62  63  64  65  64  69  74 2E 65  78  65  20 2F 64  65 6C 65  74  65  76  61 6C 75  65  20  73  61  66  65  62 6F 6F 74}  
	    $cmd5= {43 3A 5C 57  69 6E 64 6F 77  73 5C 53  79  73 4E 61  74  69  76  65 5C 62  63  64  65  64  69  74 2E 65  78  65  20 2F 64  65 6C 65  74  65  76  61 6C 75  65  20  73  61  66  65  62 6F 6F 74}  
	    $cmd6= {62  63  64  65  64  69  74  20 2F 73  65  74  20  73  61  66  65  62 6F 6F 74  20 6E 65  74  77 6F 72 6B}  
	    $cmd7= {43 3A 5C 57  69 6E 64 6F 77  73 5C 53  79  73  74  65 6D 33  32 5C 62  63  64  65  64  69  74 2E 65  78  65  20 2F 73  65  74  20  73  61  66  65  62 6F 6F 74  20 6E 65  74  77 6F 72 6B}  
	    $cmd8= {43 3A 5C 57  69 6E 64 6F 77  73 5C 53  79  73 4E 61  74  69  76  65 5C 62  63  64  65  64  69  74 2E 65  78  65  20 2F 73  65  74  20  73  61  66  65  62 6F 6F 74  20 6E 65  74  77 6F 72 6B}  
	    $cmd9= {2F 43  20  73  68  75  74  64 6F 77 6E 20 2D 72  20 2D 66  20 2D 74  20  30}  
    condition:  
	    uint16(0) == 0x5A4D  and  
	    (  
		    $ransom_note  or  
		    2 of ($string*) and  
		    3 of ($cmd*) or  
		    any of ($file*)  
	    )  
    }

More samples of Black Basta 1.0 were collected from Malware Bazaar in order to verify the YARA rule. The above YARA rule was able to detect them.

{{< image src="https://github.com/CryptoGenNepal/blog/assets/142308575/d90bd317-1e36-4f38-bbce-a900c21637d2" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}
