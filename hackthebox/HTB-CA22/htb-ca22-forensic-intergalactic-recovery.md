---
title: HTB-CA22-forensic-intergalactic-recovery
date: "2022-05-19"
description: forensic challenge during Cyber Apocalypse 2022
tldr: raid, disk reassembly
tags: [htb,hackthebox,ca22,cyber-apocalypse22,forensic,raid-reassembly]
---

## Disclaimer
I didn't solve this challenge during the CTF, i was just about to when the time ran out. But i wanted to do it because it was fun to play around with some raid-tools that i hadn't done before.

## Background
Each challenge had a story tied to it, for this challenge it was:   

Miyuki's team stores all the evidence from important cases in a shared RAID 5 disk. Especially now that the case IMW-1337 is almost completed, evidences and clues are needed more than ever.  
Unfortunately for the team, an electromagnetic pulse caused by Draeger's EMP cannon has partially destroyed the disk. Can you help her and the rest of team recover the content of the failed disk? Download: http://134.209.177.115/forensics/forensics_intergalactic_recovery.zip


## Attacking the challenge 
Download the zip file, inside it there's 3 `.img` files, where the `disk3.img` is much smaller than the other two. We'll assume that this is the broken disk.

Create loop-devices for the .img files with losetup (included in kali).

More information about losetup: [https://www.computerhope.com/unix/losetup.htm](https://www.computerhope.com/unix/losetup.htm)  

 Kali uses /dev/loop0-2 , for snapshot-reasons, so take the next in line.

```shell
sudo losetup /dev/loop3 disk1.img
sudo losetup /dev/loop4 disk2.img
sudo losetup /dev/loop5 disk3.img
```

To build a raid from those, we need a tool called `mdadm`
```shell
sudo apt install -y mdadm
```

And then to rebuild the raid, and assume that `disk3.img` is broken, but data can be rebuilt from the other two: 

```shell
sudo mdadm --create --assume-clean --level=5 --raid-devices=3 /dev/md0 /dev/loop4 missing /dev/loop3
```

Mount the `/dev/md0` disk, and copy out the pdf file we're looking for.

```shell
mkdir /mnt/tmpraid
mount /dev/md0 /mnt/tmpraid
cp /mnt/tmpraid/*.pdf .
```

Open the PDF and behold the beautiful investigation: 
![htb-ca22-igr1](/img/htb-ca22-igr1.png)


Flag: `HTB{f33ls_g00d_t0_b3_1nterg4l4ct1c_m0st_w4nt3d}`



