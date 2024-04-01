---
title: HTB-CA22-rev-wide
date: "2022-05-19"
description: rev challenge during Cyber Apocalypse 2022
tldr: reverse-engineering,ghidra
tags: [htb,hackthebox,ca22,cyber-apocalypse22,rev,reverse-engineering,ghidra]
---

## Backstory
Each challenge had a story tied to it, for this challenge it was:   

We've received reports that Draeger has stashed a huge arsenal in the pocket dimension Flaggle Alpha.  
You've managed to smuggle a discarded access terminal to the Widely Inflated Dimension Editor from his headquarters, but the entry for the dimension has been encrypted. Can you make it inside and take control?  

  
## Attack the challenge   
Because it's a `rev` challenge, open it in `ghidra` and look through the code.  
  
After looking around in the functions, this part stood out: 
  
```c
printf("[X] That entry is encrypted - please enter your WIDE decryption key: ");  
fgets(local_c8,0x10,stdin);  
mbstowcs(local_1c8,local_c8,0x10);  
iVar1 = wcscmp(local_1c8,L"sup3rs3cr3tw1d3");  
if (iVar1 == 0) {  
```

The password `sup3rs3cr3tw1d3` is what we're looking for , and after enumerating the different dimensions (trial & error) we found the encrypted entry which requires a password. 

```shell
Which dimension would you like to examine? 6  
[X] That entry is encrypted - please enter your WIDE decryption key: sup3rs3cr3tw1d3  
HTB{str1ngs_4r3nt_4lw4ys_4sc11}
```

Flag: `HTB{str1ngs_4r3nt_4lw4ys_4sc11}`