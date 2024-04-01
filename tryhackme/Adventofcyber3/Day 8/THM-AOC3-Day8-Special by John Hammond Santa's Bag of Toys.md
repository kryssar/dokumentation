---
title: Advent Of Cyber 3 - Day 8
date: "2021-12-08"
description: Forensics for Santa
tldr: Forensics
tags: [special,tryhackme,adventofcyber3,forensics]

---

Santa's laptop has been stolen, but there's some information gathered that lets us do some forensic work to see what has happened.

Start up the machine for this room and use the desktop on the side in order to find the information we're after.

Look through the logs in the folder: 
![[thm-aoc3-day8-1.png]]

Operating system on santa's laptop: `Microsoft Windows 11 Pro`

Password for the "backdoor" account: `grinchstolechristmas`

File copied to the desktop, information can be found in the largest log-file: `C:\Users\santa\AppData\Local\Microsoft\Windows\UsrClass.dat`

LOLbin , living off the land that is used to encode a file (can also be found in the larger log file): `certutil.exe`

in the log file, the attacker verifies the encoding by using `type` and with this, getting the contents of `santa.dat`. 

Copy the contents between: 
`-----BEGIN CERTIFICATE-----`
`-----END CERTIFICATE-----`

and paste into cyberchef to decode (it seems to be Base64 so try that one out). 

we are after the `UsrClass.dat` file, and to restore it, so it can be viewed with ShellBagsExplorer. 

Save the resulting file from cyberchef and open up with ShellBagsExplorer. 
![[thm-aoc3-day8-2.png]]

go through the found material and find the answers. 
What specific folder reveals that this might be a publicly available software ?: 
`.github` (found under SantaRat-main)

name of the file found in the folder with santa's toys: `bag_of_toys.zip`

google for SantaRat + github, and find the public repository: 
`https://github.com/Grinchiest/SantaRat`

check the profile of the creator: `https://github.com/Grinchiest`

Name of the owner: `Grinchiest`

interesting repo for the investigation: `[operation-bag-of-toys](https://github.com/Grinchiest/operation-bag-of-toys)`

under one of the [commits](https://github.com/Grinchiest/operation-bag-of-toys/commit/41615462e4fdc0ceeb4ef1bec693ec3de1125ed2), the password for santas packed toys is revealed: 
```
stole Santa's bag of toys!!!!!!!!!!!!!!

pw: TheGrinchiestGrinchmasOfAll
```

executable that installed a unique utility for data-exfiltration: `uharc-cmd-install.exe`

contents of malicious files: `grinchmas`

with the password found in one of the github commits, unpack the bag_of_toys.uha , and retrieve the: `228` files.


