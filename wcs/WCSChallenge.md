# Intro
A challenge built for a CTF held during an AW together with Cybix. Where the goal was to challenge them a bit when it comes to thinking outside the box and perhaps develop some other methods than the ones ingrained with nmap and such. 


# Find the port 
Running tools like `nmap` just shows all the ports as open, meaning we have no clue which ones that actually are open. 


## Python way
The information about the task said something about a webpage, so we can create a python script in order to crawl through the ports and see if atleast one of them reply with another answer. 

```python
import requests

# Find port that should reply with something
for i in range(2000, 2010, 1):
    url = "http://(IP)"
    testurl = url + ":" + str(i)
    print("trying url:" + testurl)
    
    try: 
        response = requests.get(testurl)
    #if response.status_code != 200:
    except requests.exceptions.RequestException:
        continue
    print(response.text)
```

it's a pretty slow method , but it will eventually get the result we're looking for.

(printout of result)

## Other ways 
blabla


# Qrispy 
This task was added just for fun in order to have some enumeration and steganography together with a rabbit hole. 
When checking for `robots.txt` on the page `http://(IP):2004` , we see a "disallow contact.html". 
visiting the page: `http://(IP):2004/contact.html` we see a QR code only, looking at the source code we see that it's called `whoyougonnacall.jpg`. 

Download the image and run `zbarimg whoyougonnacall.jpg` , we get the phrase `ghostbusters`

trying steghide (commands) on the file with the passphrase `ghostbusters` we get a file called `secret.txt`. Use the command `cat secret.txt` and receive the flag.

# Discover the XXE 
The page says to upload a word document, trying that out with a regular word-document containing just the text "test" inside, we get a printout on the webpage saying something about the property `title`. 

Unzip a regular word-document and look through the files for the `title` property, which is found inside the subfolder `docProps`. 

Edit the file `core.xml` and try to write something inside the property `<dc:title>foo</dc:title>` to see if that will be printed after uploading a file. If it's printed, we can do some malicious stuff, like using a XXE (link to XXE) to get local file inclusion and information disclosure by printing files like `/etc/passwd`.

## Use the XXE for username
modify word-doc `docProps/core.xml` to contain `file:///etc/passwd` , upload it and view the output. There seems to be an interesting username here other than `root` , so the user `havbroresentful` is found.

## Use the XXE for private SSH key
modify word-doc `docProps/core.xml` to contain `file:///home/havbroresentful/id_rsa.key` because the task mentioned something about leaving your private SSH keys around for anybody to read.

Upload the word-doc and save the SSH key, here it helps to view the resulting page in the mode `view-source` , so the SSH key won't get too mangled.

save the file and change the permissions: `chmod 600 id_rsa.key` 

Great, now we have a user and a SSH key, but which port is SSH running on ?

# Another agent
go through some of the regular user-agents , but the hint says something about "when you're not using curl, you're using?" and then try `wget http://(ip):2004`
and `cat index.html` , find the flag in here. 

Looking through the code, we see something about  "random port". This could very well be the SSH port, so let's try that out. 

# Access granted 
SSH in with the found user, key and port: 
`ssh -i id_rsa.key (ip) -p 2222`

# Privilege escalation 
Logged in via SSH as the `havbroresentful` user, we can use the regular `linenum.sh` and `linpeash.sh` in order to find some interesting things. 

Inside the `home` folder of the user, there's a python file called `hack.py` which includes a file called "webbrowser" , if we search the system for the file, we see that it has full privileges, and perhaps a library hijacking is possible. 

(insert printout about file permissions on webbrowser.py)

Just for fun, run sudo -L as well, and from what we can see here, the command (python path) /home/havbroresentful/hack.py can be run with root privileges.

create a file called "webbrowser.py" in the home-folder of the user , containing: 

```python
import os
os.system("/bin/bash")
```

then run `hack.py` with the sudo command. 

Congrats, you are now root ! 
