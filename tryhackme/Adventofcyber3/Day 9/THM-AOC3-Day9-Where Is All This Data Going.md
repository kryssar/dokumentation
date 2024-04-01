---
title: Advent Of Cyber 3 - Day 9
date: "2021-12-09"
description: PCAP analysis
tldr: Networking
tags: [pcap,tryhackme,adventofcyber3,networking]

---

Open up the file: `AoC3.pcap` in wireshark and start analysing. 

filter for first question about what folder that was found:  `http.request.method == GET` and the folder is `login`

filter for POST question, username and password used: `http.request.method == POST`

follow TCP-stream to find information about username and password: `username=McSkidy&password=Christmas2021%21`
URL decoded: `username=McSkidy&password=Christmas2021!`
answer: `McSkidy:Christmas2021!`

filter: tcp.stream eq 2
flag found in user-agent: `User-Agent: TryHackMe-UserAgent-THM{d8ab1be969825f2c5c937aec23d55bc9}`

TXT record filter: `udp.port == 53` and scroll down to find a TXT record , which is `packet.tryhackme.com`
follow UDP stream and find flag: `AoC3 is awesome - THM{dd63a80bf9fdd21aabbf70af7438c257}`

to find FTP user, search for `ftp-data`

follow TCP stream. 
```
220 (vsFTPd 3.0.3)
USER tryhackftp
331 Please specify the password.
PASS TryH@ckM3!
230 Login successful.
CWD /files
250 Directory successfully changed.
PASV
227 Entering Passive Mode (10,10,10,4,210,245).
STOR secret.txt
150 Ok to send data.
226 Transfer complete.
```

the FTP password is: `TryH@ckM3!`

command used to upload the secret.txt: `STOR`

to find the contents of secret.txt, search for ftp-data and follow the tcp-stream that is related to the `STOR` command. Filter: `tcp.stream eq 6`

content: `AoC Flag: 123^-^321`

answer for content: `123^-^321`