---
title: Advent Of Cyber 3 - Day 12
date: "2021-12-12"
description: MSSQL and relational databases
tldr: Networking
tags: [mssql,tryhackme,adventofcyber3,networking]

---

Time to attack the grinchen hackers in `Grinchen Enterprises`. 
Target: `10.10.4.103` 

# Nmap scan
```shell 
sudo nmap -sC -sV -p- -Pn -vvv 10.10.4.103

Completed NSE at 16:50, 0.00s elapsed                                                                                                         
Nmap scan report for 10.10.4.103                                                                                                              
Host is up, received user-set (0.053s latency).                                                                                               
Scanned at 2021-12-12 16:43:50 CET for 400s                                                                                                   
Not shown: 65526 filtered tcp ports (no-response)                                                                                             
PORT      STATE SERVICE       REASON          VERSION                                                                                         
22/tcp    open  ssh           syn-ack ttl 127 OpenSSH for_Windows_7.7 (protocol 2.0)                                                          
| ssh-hostkey:                                                                                                                                
|   2048 b8:0b:21:22:72:ce:77:c5:b2:1f:e6:ad:80:48:5c:a7 (RSA)         
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCpiyWOix6MKHshmfY385ynnWTavZm9vUA0hRazTpDjRgEeLO/4EaTRDLw9zaiJD3H0dR7btlfGiwmibakoXX1hOFEN4GIcGj2thhYf
3lOwgCuoswN/u2DInJXTElMdCGGSqY95/IkHhFK+jAFS2E04IKnFI5S9hompr2NXa8NCnmNjMAdmtUJ6xm1ZlTAZ4EB6/IziWQVyHraQxsZG6XuTY/m+jWHRUwxmEl3jNOk/eQEYFwsLV0
BxPSYPTrkyo0vR3ku/FMc36S5fw1rauaGF3wzxWvysYrAOfJdev5cVIsrY157oyxV0BZJGFmhGi3Nyb/lXjIA2B/r7R9QDaofh                                            
|   256 56:dc:0c:78:a1:97:40:ad:b7:78:f8:72:44:97:bc:96 (ECDSA)        
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLRLOBuaWrIJsW0jMabUiw76xs2j/OTetgembgS8rfGYLTnv34N1bNOd7QmyUrBbxL1D
g62pIUmgqfwmvJBH584=               
|   256 1c:ac:1b:ea:2a:e0:cd:b0:47:48:96:c6:4d:73:47:25 (ED25519)      
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILflLG8U4fEarBRhugxgC48YrLHoTVtwAqmIwq4Bt10r 
111/tcp   open  rpcbind       syn-ack ttl 127 2-4 (RPC #100000)        
| rpcinfo:                         
|   program version    port/proto  service                             
|   100000  2,3,4        111/tcp   rpcbind                             
|   100000  2,3,4        111/tcp6  rpcbind                             
|   100000  2,3,4        111/udp   rpcbind                             
|   100000  2,3,4        111/udp6  rpcbind                             
|   100003  2,3         2049/udp   nfs                                 
|   100003  2,3         2049/udp6  nfs                                 
|   100003  2,3,4       2049/tcp   nfs                                 
|   100003  2,3,4       2049/tcp6  nfs                                 
|   100005  1,2,3       2049/tcp   mountd                              
|   100005  1,2,3       2049/tcp6  mountd                              
|   100005  1,2,3       2049/udp   mountd                              
|   100005  1,2,3       2049/udp6  mountd                              
|   100021  1,2,3,4     2049/tcp   nlockmgr                            
|   100021  1,2,3,4     2049/tcp6  nlockmgr                            
|   100021  1,2,3,4     2049/udp   nlockmgr                            
|   100021  1,2,3,4     2049/udp6  nlockmgr                            
|   100024  1           2049/tcp   status                              
|   100024  1           2049/tcp6  status                              
|   100024  1           2049/udp   status                              
|_  100024  1           2049/udp6  status                              
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC                                                                           
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn                                                                   
445/tcp   open  microsoft-ds? syn-ack ttl 127                          
2049/tcp  open  mountd        syn-ack ttl 127 1-3 (RPC #100005)   
3389/tcp  open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services                                                                     
|_ssl-date: 2021-12-12T15:49:56+00:00; -6s from scanner time.          
| ssl-cert: Subject: commonName=AOC2021-NFS                            
| Issuer: commonName=AOC2021-NFS   
| Public Key type: rsa             
| Public Key bits: 2048            
| Signature Algorithm: sha256WithRSAEncryption                         
| Not valid before: 2021-11-05T08:54:44                                
| Not valid after:  2022-05-07T08:54:44                                
| MD5:   67de dffc e00a 7c07 8fc3 d895 27ce 62b1                       
| SHA-1: 8b28 8c49 7f3f 2323 244b 9eaa 9bce 6e4e 453b 7e9a             
| -----BEGIN CERTIFICATE-----      
| MIIC2jCCAcKgAwIBAgIQdBwWiBEF/rZE5PUfpBXPaTANBgkqhkiG9w0BAQsFADAW     
| MRQwEgYDVQQDEwtBT0MyMDIxLU5GUzAeFw0yMTExMDUwODU0NDRaFw0yMjA1MDcw     
| ODU0NDRaMBYxFDASBgNVBAMTC0FPQzIwMjEtTkZTMIIBIjANBgkqhkiG9w0BAQEF     
| AAOCAQ8AMIIBCgKCAQEAsAQLACQoKhjag3rm5d4n7WMO7AZxHFvM/PyD1+bGA5/g     
| bpokRQre0B5oL9HEkrflWWZnzWojOSllM8ufnJMiIJvGxi0GgzOzVuYYw2FAJnkQ     
| j+cJ4Eo4h/LF7+JZLkXvPKTgZ4hjhK+puMeed3DrEXxlNpDR9zbqz2VPT7/YRmXK     
| MO7QKiqGdGBmxTm90ssxaR2OhJor/32LBJxo+jLQcc4OjZsWoAbeBUYgADOQDGWK     
| KcmWBw5h3O04dkuUcRYlPrFjCoVqkPJk3F3pv/Gs2EqLXtK5YzHdrPpvE4oVcRvq     
| NqbiLtcDbdEvcozmpDh88KR7kh9Lpmv0mjKirLrazQIDAQABoyQwIjATBgNVHSUE     
| DDAKBggrBgEFBQcDATALBgNVHQ8EBAMCBDAwDQYJKoZIhvcNAQELBQADggEBAKvb     
| onb+4Y6mY2lPrtNEp4jA2VOJjX9AtMBQnwLEokXnRwvNqTRX84hNMZarCW2pvKCa     
| BdJWImkoLmnbpt8mNNrk20R1f1I9FzF/HpL0Qbn7iJbErPqWE3QZ4niCvL9wHqgO     
| Wx/YX1LiNPipWpCXqvkfnA0Grk9x5+sXR9pMpVdgWkP22wiQQ1GL2vbwSnDZzwiu     
| SfZinTfS4Q8EeXx/6Y9PowvWXdHRgzq68cnozswrGno1dfKHz/fpSuE+vdzg+9B9     
| +mvd0iQX3oK4YbyFxADXX62ecggTba+GbJcoDLJDwg3Bf/8R1wHOINor3BoCUQ2y     
| FFsSszdwgMAQJcYeCmo=             
|_-----END CERTIFICATE-----
| rdp-ntlm-info:                   
|   Target_Name: AOC2021-NFS       
|   NetBIOS_Domain_Name: AOC2021-NFS                                   
|   NetBIOS_Computer_Name: AOC2021-NFS                                 
|   DNS_Domain_Name: AOC2021-NFS   
|   DNS_Computer_Name: AOC2021-NFS                                     
|   Product_Version: 10.0.17763    
|_  System_Time: 2021-12-12T15:49:17+00:00                             
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC                                                                           
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC                                                                           
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows               

Host script results:               
|_clock-skew: mean: -5s, deviation: 0s, median: -6s                    
| smb2-security-mode:              
|   3.1.1:                         
|_    Message signing enabled but not required                         
| p2p-conficker:                   
|   Checking for Conficker.C or higher...                              
|   Check 1 (port 59674/tcp): CLEAN (Timeout)                          
|   Check 2 (port 22188/tcp): CLEAN (Timeout)                          
|   Check 3 (port 55202/udp): CLEAN (Timeout)                          
|   Check 4 (port 2265/udp): CLEAN (Timeout)                           
|_  0/4 checks are positive: Host is CLEAN or ports are blocked        
| smb2-time:                       
|   date: 2021-12-12T15:49:19      
|_  start_date: N/A                


scan number 2:
$ sudo nmap -sS -Pn 10.10.4.103             
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-12 16:53 CET
Nmap scan report for 10.10.4.103
Host is up (0.057s latency).
Not shown: 993 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
111/tcp  open  rpcbind
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 5.24 seconds

```

Open ports: `7`
Port usually used by mountd: `2049`

explore NFS by using `showmount` , and show the exported volumes with the option: `-e`

```shell
$ showmount -e 10.10.4.103
Export list for 10.10.4.103:
/share        (everyone)
/admin-files  (everyone)
/my-notes     (noone)
/confidential (everyone)
```

there are some folders that `everyone` can read, which never is a good idea.

shares found: `4`
shares with everyone: `3`

because everyone can read the shares, we as attackers can mount them on our own attacking box.

example: 
```shell
mkdir -p /mnt/victimfolder
mount 10.10.4.103:/(sharename) /mnt/victimfolder
```

for day 12: 
```shell
$ sudo mkdir -p /mnt/day12
$ sudo mkdir -p /mnt/day12/{share,admin-files,confidential}

$ sudo mount 10.10.4.103:/share /mnt/day12/share
$ sudo mount 10.10.4.103:/admin-files /mnt/day12/admin-files <- not permitted
$ sudo mount 10.10.4.103:/confidential /mnt/day12/confidential
```

then go through the mounted folders and see if there's any interesting information. 

jackpot, found ssh keys: 
```shell
┌──(kryssar㉿kali)-[/mnt/day12]
└─$ ls confidential 
ssh
   
┌──(kryssar㉿kali)-[/mnt/day12]
└─$ ls confidential/ssh 
id_rsa  id_rsa.pub
```

title of 2680-0.txt: `Meditations` (located under `/share`)
name of share with SSH keys: `confidential`

MD5 sum of `id_rsa`:  `3e2d315a38f377f304f5598dc2f044de`
```shell
$ md5sum id_rsa        
3e2d315a38f377f304f5598dc2f044de  id_rsa
```