---
title: Advent Of Cyber 3 - Day 10
date: "2021-12-10"
description: NMAP scanning
tldr: Networking
tags: [nmap,tryhackme,adventofcyber3,networking]

---

Start up the attached machine, connect OpenVPN to the tryhackme network and fire up nmap on the target(s).

Target IP: `10.10.253.217`

```shell
sudo nmap -sT 10.10.253.217
[sudo] password for kryssar: 
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-12 15:27 CET
Verbosity Increased to 1.
Verbosity Increased to 2.
Completed Connect Scan at 15:27, 1.96s elapsed (1000 total ports)
Nmap scan report for 10.10.253.217
Host is up (0.043s latency).
Scanned at 2021-12-12 15:27:25 CET for 2s
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 2.14 seconds
           Raw packets sent: 4 (152B) | Rcvd: 1 (28B)

```

amount of ports open: `2`
smallest port number open: `22`
service related to higher number: `http`

```shell
$ sudo nmap -sS 10.10.253.217                                                                        127 ⨯
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-12 15:29 CET
Nmap scan report for 10.10.253.217
Host is up (0.089s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.99 seconds
```

did we get the same result: `Y`

version of services can be detected with: `nmap -sV (ip)`

```shell
$ sudo nmap -sV 10.10.253.217
[sudo] password for kryssar: 
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-12 15:44 CET
Nmap scan report for 10.10.253.217
Host is up (0.047s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.49
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.76 seconds
```

version of webserver: `Apache httpd 2.4.49`

CVE for the apache version can be found with searchsploit: 
```shell
searchsploit apache 2.4.49

Apache HTTP Server 2.4.49 - Path Traversal & Remote Code Execution (RCE)   | multiple/webapps/50383.sh
```

[Apache vulnerabilities for 2.4](https://httpd.apache.org/security/vulnerabilities_24.html)

version of CVE that got fixed in 2.4.51: [CVE-2021-42013](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-42013)

find more open ports by scanning all of them instead of the top 1000. 

```shell 
$ sudo nmap -sS -p- 10.10.253.217                                                                      1 ⨯
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-12 15:53 CET
Nmap scan report for 10.10.253.217
Host is up (0.046s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
20212/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 22.50 seconds
```

new open port found: `20212`

```shell
$ sudo nmap -sV -p20212 10.10.253.217
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-12 15:55 CET
Nmap scan report for 10.10.253.217
Host is up (0.045s latency).

PORT      STATE SERVICE VERSION
20212/tcp open  telnet  Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.60 seconds
```

name of service running on that port: `telnetd`



