---
title: Forensic Challenges Writeups | Nahamcon CTF 2023
tags: CTF Forensics
---

We, bER4bb1t$ team, are glad to secure the 7th with tie on the 6th rank with TCP1P team in nahamcon CTF 2023, actually our team did a great stuff.
Here're our writeups for forensics challenges.


# Perfectly Infected (Easy, 604 Solves)
Ever had to deal with dirty files? No more! Our patented technology will make your files sparkling clean! Sample included. 

## Solution:
It's a pdf file and need the flag, so easily have used pdf-parser tool.
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic1.png)
```flag{b00acdc78749b378f8f4889f8243789304abe928}```


# Fetch (Easy, 166 Solves)
"Gretchen, stop trying to make fetch happen! It's not going to happen!" - Regina George 

## Solution:
After extracting the compressed file, it's WIM image file.
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic2.png)
A WIM (Windows Imaging Format) file is a file-based disk image format used by Microsoft Windows. It is commonly used for capturing and deploying system images, including the operating system, applications, and data. WIM files are designed to be highly compressed and support file-based imaging, allowing individual files to be added, modified, or removed from the image without having to recreate the entire image.

WIM files can be mounted as virtual drives, allowing users to access and modify the contents of the image. They can also be used for offline servicing, enabling administrators to apply updates, install applications, or make configuration changes to the captured system image.

Overall, they provide a flexible and efficient way to create and manage system images in Windows environments, making them a popular choice for system deployment, recovery, and maintenance tasks.

Let's try to mount it:
- Run cmd as administrator
- ```mkdir fetch_output_dir```
- ```dism /mount-wim /wimfile:D:\CTF\nahamcon\fetch /index:1 /mountdir:D:\CTF\nahamcon\fetch_output_dir```

As a result, list of prefetch files are exported as below.
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic3.png)

Ok, it's time to use zimmerman's tool, PECmd.exe (prefetch explorer).

```PECmd.exe -d D:\CTF\nahamcon\fetch_output_dir | findstr /i "flag"```
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic4.png)
```flag{97f33c9783c21df85d79d613b0b258bd}```


# Raided (Medium, 61 solves)
The police raided a server belonging to a very 1337 hax0r that was used to stage attacks. Upon further investigation, this server turned out to be a jump server for the attacker to access more infrastructure.
A memory snapshot was taken of the machine. See if you can figure out what the attacker was doing and what other systems the hacker was accessing. 
## Solution

It's a memory image for a jump server, so we should search for any remote connection artifacts to trace the incident.
I've imported the image into volatility but didn't be recognized, hence I've used the power of strings & grep =D.

```strings raided-challenge-dump-vmem| grep ssh ```
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic5.png)
We've found the a user and host, and interestingly authentication is done using explicit private key.

```strings raided-challenge-dump-vmem| grep "BEGIN " -A 20```
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic6.png)
Fortunately, an openssh private key was found, so I had to try it.
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic7.png)
```flag{654e9dc4c424e25423c19c5e64fffb27}```


# IR (challenge group)
It's an OVA file, and has multiple questions.

# IR#1 (Easy, 141 solves)
Can you find the hidden file on this VM?

## Solution
I've parsed the VM disk after extracting it into autopsy for fast insights.
After some search for suspicious hidden files.
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic8.png)
```flag{053692b87622817f361d8ef27482cc5c}```


# IR#2 (Easy, 141 solves)
Can you figure out how the malware got onto the system? 
## Solution
There're a suspicious obfuscated powershell script in the Downloads folder called updates.ps1.
Two of the most common initial access methods are drive-by-comprise and phishing. Had search for any artifacts related to browser activities but nothing related the malicious script.

So, I had to check to any information about delivered mails, so I simply opened the mail client and found the below mail:
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic9.png)
```flag{75f086f265fff161f81874c6e97dee0c}.```

# IR#3 (Medium, 151 solves)
Can you reverse the malware? 
## Solution
To reverse the malware we have to options:

- Deobfuscate the script and figure out the source code function
- Suppose script execution, and check powershell script block logs 

First, I had to check the second way with hope to find script block logging enabled, the easiest.
By default, powershell script block logging is stored in file ```C:\Windows\System32\winevt\Logs\Microsoft-Windows-PowerShell%4Operational.evtx``` and we're searching for event id ```4104```.

So, I've written the following powershell command after extracting the log file I found.

```Get-WinEvent -Path "Microsoft-Windows-PowerShell%4Operational.evtx" -FilterXPath "*[System[EventID=4104]]" | ForEach-Object { $_.ToXml() } ```

After searching in the output, I've found the obfuscated script and its execution so, hence a plaintext version appeared. Now we can know the malware's functionalities.
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic10.png)

```flag{892a8921517dcecf90685d478aedf5e2}```

IR#4 (Medium, 155 solves)
Where is the data being exfiltrated? Please give the MD5 hash of the URL with the usual wrapper of flag{}.

Simple enough, from the above screenshot we can make sure that the script makes post requests to url ( https://www.thepowershellhacker.com/exfiltration ) as a method of data exfiltration.

flag{32c53185c3448169bae4dc894688d564}


# IR#5 (Medium, 127 solves)
Can you please recover our files? 
## Solution
We can write a decryptor, but I prefer the lazy way again,  just recovering them by autopsy and grep for potential flag format.
![](/assets/images/NahamCon_CTF_2023_Forensics_Writeups/pic11.png)
``flag{593f1527d6b3b9e7da9bdc431772d32f}```


Thanks for reading <3.
![image](https://github.com/0xOFFSET/0xOFFSET.github.io/assets/19360052/99be2058-f30f-4690-8a19-ee98af0356f4)
