---
title: Advent Of Cyber 3 - Day 20
date: "2021-12-20"
description: File analysis
tldr: File analysis
tags: [virustotal,tryhackme,adventofcyber3,blueteam]

---

Today a couple of tools like `file` , `strings` and `md5sum` were used to analyze files and their behaviour, combined with [virustotal](https://www.virustotal.com)

output after running `strings` on `testfile` :
`X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*`

running `file` on `testfile` :
`EICAR virus test files`

Calculate file hash with `md5` and search for it on virustotal, when did it first appear in the wild ? : 
`2005-10-17 22:03:48`

What is the assigned classification from Microsoft ? : 
`Virus:DOS/EICAR_Test_File`

what are the two first names of the EICAR file ? : 
`ducklin.htm or ducklin-html.htm`

what is the maximum number of characters that can be in the file ? : `128`

EOF