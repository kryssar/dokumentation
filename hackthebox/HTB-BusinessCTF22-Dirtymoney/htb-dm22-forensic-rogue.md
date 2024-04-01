---
title: HTB-DirtyMoney2022-forensic-rogue
date: "2022-07-19"
description: forensic challenge during Business CTF - Dirty Money 2022
tldr: pcap, minidump, mimikatz, SMBv3 decryption, wireshark
tags: [htb,hackthebox,dm22,dirty-money22,forensic,wireshark,mimikatz,minidump,pcap, smb-decrypt]
---

## Backstory
Each challenge had a backstory tied to it, for this one it was: 

```
SecCorp has reached us about a recent cyber security incident.  
They are confident that a malicious entity has managed to access a shared folder that stores confidential files.  
Our threat intel informed us about an active dark web forum where disgruntled employees offer to give access to their employer's internal network for a financial reward.  
In this forum, one of SecCorp's employees offers to provide access to a low-privileged domain-joined user for 10K in cryptocurrency. Your task is to find out how they managed to gain access to the folder and what corporate secrets did they steal.
```


## Attack the challenge
Download the pcap-file and start analyzing it with `protcol hierarchy statistics` in Wireshark.  

**obs! Make sure to use the latest wireshark for this challenge!**

There are a couple of things that stand out, but for now we'll begin with the FTP parts.

filtering for just ftp and following the TCP-stream results in: 
```shell
220 (vsFTPd 3.0.3)
USER ftpuser
331 Please specify the password.
PASS SZC0aBomFG
230 Login successful.
OPTS utf8 on
200 Always in UTF8 mode.
PWD
257 "/" is the current directory
TYPE I
200 Switching to Binary mode.
PASV
227 Entering Passive Mode (77,74,198,52,226,112).
STOR 3858793632.zip
150 Ok to send data.
226 Transfer complete.
```

A file is being transferred and because we have the pcap, we can extract the `3858793632.zip` file. 

Filter on `ftp-data` and dig a bit further by following the TCP-stream (after filtering).

change some parameters: 
`show data as: raw`

then `save as` and to maintain the same name as in the pcap-transfer, save it as: `3858793632.zip`  

Inside the ZIP-file is another file called: `3858793632.pmd` , which (according to Kali) is a minidump. 

```shell
$ file 3858793632.pmd 
3858793632.pmd: Mini DuMP crash report, 13 streams, Mon Jul  4 11:39:18 2022, 0x6 type
```

Me and my colleagues tried digging around and analyzing the minidump with a number of different tools in order to try and get something out of it. 

Some of the things we tried: 
```shell
WinDbg
xxd
hexdump
hexeditor
strings
HxD
MinidumpExplorer
Bluescreenview
whocrashed
```

But that really didn't get us anywhere, until a colleague found out that there's a portion of SMBv3 traffic in the pcap, which is encrypted.

And going by this [link](https://medium.com/maverislabs/decrypting-smb3-traffic-with-just-a-pcap-absolutely-maybe-712ed23ff6a2), there's a script that can be used. Provided that we have all the information required of course.

So we had some of the information from the pcap , but we needed the user, domain, ntlm hash  in order to get further. 

Luckily [mimikatz](https://abawazeeer.medium.com/using-mimikatz-to-get-cleartext-password-from-offline-memory-dump-76ed09fd3330) can help with this task.

```shell
mimikatz # sekurlsa::minidump 3858793632.pmd  
Switch to MINIDUMP : '3858793632.pmd'    
mimikatz # sekurlsa::logonPasswords full
```

The user that is initiating the SMBv3 traffic is mentioned in the pcap as `athomson`.  Which can be seen in the packet with `session id`. 

```
Session Id: 0x0000a00000000015 Acct:athomson Domain:CORP Host:WS02
```

grabbing that information from the mimikatz dump: 
```
Authentication Id : 0 ; 3857660 (00000000:003adcfc)
Session           : RemoteInteractive from 2
User Name         : athomson
Domain            : CORP
Logon Server      : CORP-DC
Logon Time        : 2022-07-04 13:32:10
SID               : S-1-5-21-288640240-4143160774-4193478011-1110
        msv :
         [00000003] Primary
         * Username : athomson
         * Domain   : CORP
         * NTLM     : 88d84bad705f61fcdea0d771301c3a7d
         * SHA1     : 60570041018a9e38fbee99a3e1f7bc18712018ba
         * DPAPI    : 022e4b6c4a40b4343b8371abbfa9a1a0
        tspkg :
        wdigest :
         * Username : athomson
         * Domain   : CORP
         * Password : (null)
        kerberos :
         * Username : athomson
         * Domain   : CORP.LOCAL
         * Password : (null)
        ssp :
        credman :
        cloudap :       KO

```

Variables found: 
```shell
user: athomson
domain: CORP
ntlm hash: 88d84bad705f61fcdea0d771301c3a7d
session_id: 0x0000a00000000015
```

Next up we need to find the `sessionkey` and another variable called `ntproofstr`. 

The easiest way to filter those out was with `tshark`

```shell
tshark -r capture.pcapng -Y smb2 -T fields -e smb2.sesid  
0x0000a00000000015  
  
tshark -r capture.pcapng -Y smb2 -T fields -e ntlmssp.auth.sesskey  
032c9ca4f6908be613b240062936e2d2  
28be9df22813cdfa83d25bf08b63049f  
  
tshark -r capture.pcapng -Y smb2 -T fields -e ntlmssp.ntlmv2_response.ntproofstr  
d047ccdffaeafb22f222e15e719a34d4  
d09104b2ad7feed3c5e9c30dcb444553
```

now all variables should be gathered and we can continue by running the script to get the actual `random key` in order to decrypt SMBv3. 

We had some issues with the script and didn't get the correct key until it was too late. 

The kind person [godylockz](https://gist.github.com/godylockz/71837f139bce120e12d8f7bf9f34d477) made an updated version of the script which got us on the right track, but after the competition was done

```shell
python calc_smb_key.py -u athomson -d CORP -ph 88d84bad705f61fcdea0d771301c3a7d -n d047ccdffaeafb22f222e15e719a34d4 -k 032c9ca4f6908be613b240062936e2d2 -i 0000a00000000015  
ID: 1500000000a00000  
Random SK: 9ae0af5c19ba0de2ddbe70881d4263ac
```

When the ID and Random SK are correct, we can enter it into wireshark and get the SMBv3 packets decrypted.

Wireshark -> Edit -> Preferences -> Protocols -> SMB2 

here there should be an `Edit` button right after the text: `Secret session keys for decryption` , click it and then enter the information: 
![htb-dm22-forensic-rogue](htb-dm22-rogue1.png)

After accepting/closing the dialog-boxes, wireshark will start to decrypt the SMBv3 messages.
![htb-dm22-rogue](htb-dm22-rogue3.png)

Now we can export the file which was transfered. 
Wireshark -> File -> Export Objects -> SMB   
![htb-dm22-rogue](htb-dm22-rogue2.png)

opening up the PDF file and looking at page #3 , we finally get the flag. 

![htb-dm22-rogue](htb-dm22-rogue4.png)

Flag: `HTB{n0th1ng_c4n_st4y_un3ncrypt3d_f0r3v3r}`


## EOF
