---
title: HTB-CA22-forensic-seized
date: "2022-05-19"
description: forensic challenge during Cyber Apocalypse 2022
tldr: pcap, dns, deobfuscation, powershell, aes decryption
tags: [htb,hackthebox,ca22,cyber-apocalypse22,forensic,powershell,deobfuscation,pcap, aes-decrypt, dns]
---

## Backstory
Each challenge had a story tied to it, for this challenge it was:   

Miyuki is now after a newly formed ransomware division which works for Longhir. This division's goal is to target any critical infrastructure and cause financial losses to their opponents.  
They never restore the encrypted files, even if the victim pays the ransom. This case is the number one priority for the team at the moment.  
Miyuki has seized the hard-drive of one of the members and it is believed that inside of which there may be credentials for the Ransomware's Dashboard.  
Given the AppData folder, can you retrieve the wanted credentials? Download: http://134.209.177.115/forensics/forensics_seized.zip


## Attack the challenge
Download and unpack the file, it's an AppData folder, which will take some time to sift through.

The google chrome folder looked juicy at a first glance so in order to help out with checking the cache, we'll take some inspiration from: 
[https://techverse.net/view-extract-files-google-chromes-cache/](https://techverse.net/view-extract-files-google-chromes-cache/)  
  
need to find a dashboard , with credentials and they could be hidden inside the AppData folder , try the chrome cache to see if there's any trails:  
  
this folder looks interesting: ls `"AppData/Local/Google/Chrome/User Data/Default"`
  
```shell
.\ChromeCacheView.exe -folder '.\AppData\Local\Google\Chrome\User Data\Default\Cache\'
```

With the help of the tool, we find a reference to `draeglocker`  
```
Filename URL Content Type File Size Last Accessed Server Time Server Last Modified Expire Time Server Name Server Response Web Site Frame Content Encoding Cache Name Cache Control ETag Server IP Address URL Length Deleted File  
dashboard.draeglocker.com [http://dashboard.draeglocker.com](http://dashboard.draeglocker.com) 0 2022-03-22 15:23:06 2022-03-22 15:23:06 HTTP/1.1 404 Not Found [http://draeglocker.com](http://draeglocker.com) [http://draeglocker.com](http://draeglocker.com) 192.168.1.8 32 No
```
  
user found with SQLITEBROWSER:  `ransomoperator@draeglocker.com ` 
  
under:` AppData/Local/Google/Chrome/User Data/Default` there was a file called `Login Data` containing the user info and a password BLOB, that can be exported as a BIN.

There were some interesting entries in the `History` file as well. 
```
https://windowsliveupdater.com/#/login
Hak5 Cloud C² 1 0 13292435709166505 0
```

Digging further into the files in google chrome, we find the file: `AppData\Local\Google\Chrome\User Data\Local State`

Which contains an interesting part.
```
os_crypt":{"encrypted_key":"RFBBUEkBAAAA0Iyd3wEV0RGMegDAT8KX6wEAAACm51uGPIZzTayfIz+HNAidAAAAAAIAAAAAABBmAAAAAQAAIAAAACc7RsTHfaauxrhBBjIqqmhrpu4YgBuonvNnS6mwHh46AAAAAA6AAAAAAgAAIAAAAL/cUy0IhgQQbDrc+KvOqsr+VCQsd9QUwZOC0v962Hf0MAAAAEHBCEaKa1Z9JzasA7wpTHI5PjeCJgrNbSTeklRxKbLst8qd8SnSo9hCOn5xwIOhwkAAAAA6QhhJeJDGW4UU26/TX3q4czhgLkuzjqXFgeH+CHdrTLjkK90vaEpXJerbw41eqFYSlsouQspBo/5R0HYeX295"}
```

if we only could decrypt the stuff somehow, hashid can't recognize the type of hash.

After some research and digging around, it looks like a `DPAPI` key. And to get the hash out, we can use a `john` python script:
[https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py](https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py)

```shell
python3 DPAPImk2john.py -S S-1-5-21-3702016591-3723034727-1691771208-1002 -mk AppData/Roaming/Microsoft/Protect/S-1-5-21-3702016591-3723034727-1691771208-1002/865be7a6-863c-4d73-ac9f-233f8734089d -c local  
$DPAPImk$2*1*S-1-5-21-3702016591-3723034727-1691771208-1002*aes256*sha512*8000*a17612a0ebdfc203316e0c18c04729f1*288*7dd629ab5efc8442596e5fbe5b9fc695bf8a51384dfacabd7a1a214245f894383540eb3e00c009bd76f836ae991cef540d74c0a6a31527b7e1df4b0d55a6760e41271f3dcaad163a6fb648f898281424728485335676c0374735cab055088e66bc55a72fc2087d64038d1d716f5efd4bdd4ce19971d082db004a36de70c351a2bd9b6ba9cf8f89a7481150b26f5808bc
```

A good resource for some more information on DPAPI hacking: 
[https://www.synacktiv.com/ressources/synacktiv_DPAPI_Sthack.pdf](https://www.synacktiv.com/ressources/synacktiv_DPAPI_Sthack.pdf)

Use hashcat to crack the hash, and the mode is `15900` , not `15300` as i first tried with and didn't get through properly. 

```
.\hashcat.exe -m 15300 -a 0 .\hash.txt .\rockyou.txt

$DPAPImk$2*1*S-1-5-21-3702016591-3723034727-1691771208-1002*aes256*sha512*8000*a17612a0ebdfc203316e0c18c04729f1*288*7dd6  
29ab5efc8442 2  
596e5fbe5b9fc695bf8a51384dfacabd7a1a214245f894383540eb3e00c009bd76f836ae991cef540d74c0a6a31527b7e1df4b0d55a6760e41271f3d  
caad163a6fb6 6  
48f898281424728485335676c0374735cab055088e66bc55a72fc2087d64038d1d716f5efd4bdd4ce19971d082db004a36de70c351a2bd9b6ba9cf8f  
89a7481150b2 2  
6f5808bc:ransom
```

The password was ‘ransom’ , how original -.-  

so far we have: 
```
user: ransomoperator@draeglocker.com  
pass: ransom
```

path to DPAPI masterkey, which we want to unlock:  
```  
AppData/Roaming/Microsoft/Protect/S-1-5-21-3702016591-3723034727-1691771208-1002/865be7a6-863c-4d73-ac9f-233f8734089d
```

another good resource for more information:  
[https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/dpapi-extracting-passwords](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/dpapi-extracting-passwords)

download mimikatz:  
[https://github.com/gentilkiwi/mimikatz/releases](https://github.com/gentilkiwi/mimikatz/releases)


```powershell 
mimikatz # dpapi::chrome /in:"AppData\Local\Google\Chrome\User Data\Default\Login Data" /masterkey:138f089556f32b87e53c5337c47f5f34746162db7fe9ef47f13a92c74897bf67e890bcf9c6a1d1f4cc5454f13fcecc1f9f910afb8e2441d8d3dbc3997794c630  
> Encrypted Key found in local state file  
> Encrypted Key seems to be protected by DPAPI  
* masterkey : 138f089556f32b87e53c5337c47f5f34746162db7fe9ef47f13a92c74897bf67e890bcf9c6a1d1f4cc5454f13fcecc1f9f910afb8e2441d8d3dbc3997794c6  
30  
> AES Key is: 46befddb52a607c5e775b7a930b6b6c4f3a35e7c1c30aaa4ce0d2277fbca6c19  
  
URL : https://windowsliveupdater.com/ ( https://windowsliveupdater.com/ )  
Username: ransomoperator@draeglocker.com  
* using BCrypt with AES-256-GCM  
Password: HTB{Br0ws3rs_C4nt_s4v3_y0u_n0w}
```

Flag: `HTB{Br0ws3rs_C4nt_s4v3_y0u_n0w}`