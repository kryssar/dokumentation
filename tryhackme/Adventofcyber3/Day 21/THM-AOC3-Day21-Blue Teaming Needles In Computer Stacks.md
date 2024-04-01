---
title: Advent Of Cyber 3 - Day 21
date: "2021-12-21"
description: Yara rules
tldr: Yara
tags: [yara,tryhackme,adventofcyber3,blueteam]

---

When analyzing files, it can be handy to verify them agains a couple of antivirus engines as well as other available tools that can analyze the behaviour. 

Yara is a tool where the community can commit rules they've created for known malware, which helps the blueteam to know what to look for. 

recommended github repo: [awesome-yara](https://github.com/InQuest/awesome-yara)

recommended THM room for playing around with yara more: [yara-room](https://tryhackme.com/jr/yara)


## todays challenge

We changed the text in the string $a as shown in the eicaryara rule we wrote, from X5O to X50, that is, we replaced the letter O with the number 0. The condition for the Yara rule is $a and $b and $c and $d. If we are to only make a change to the first boolean operator in this condition, what boolean operator shall we replace the 'and' with, in order for the rule to still hit the file? 

answer: `or`

What option is used in the Yara command in order to list down the metadata of the rules that are a hit to a file?

answer: `-m`

What section contains information about the author of the Yara rule?
answer: `metadata`

What option is used to print only rules that did not hit?
answer: `-n`

Change the Yara rule value for the $a string to X50. Rerun the command, but this time with the -c option. What is the result?

answer: `0`


EOF