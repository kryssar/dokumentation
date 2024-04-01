---
title: Yara
date: "2021-12-22"
description: Yara rules
tldr: Yara
tags: [yara,tryhackme,blueteam,forensics]

---

## Task 1 - Introduction 

Just read the provided information about Yara (Yet Another Ridiculous Acronym)

## Task 2 - What is Yara ?

Make sure that yara is installed locally as well, so we can play around more further on.

```shell 
sudo apt install -y yara 
```

link to malware room, could be interesting to investigate further on: [malmalintroductory](https://tryhackme.com/room/malmalintroductory)

What is the name of the base-16 numbering system that Yara can detect ? : `Hex`

Would the text "Enter your Name" be a string in an application? : `Yay`

## Task 3 - Installing Yara 

Oops, guess i was a bit ahead , well then. In case Yara isn't in your dist-repo , you can build from source: 

```shell
sudo apt update -y && sudo apt upgrade -y

sudo apt install automake libtool make gcc flex bison libssl-dev libjansson-dev libmagic-dev pkg-config
```

get the latest [yara](https://github.com/VirusTotal/yara/releases) release (4.1.3 currently 2021-12-22)

```shell
wget https://github.com/VirusTotal/yara/archive/refs/tags/v4.1.3.tar.gz

tar -zxvf v4.0.2.tar.gz
```

compile and install
```shell
cd yara-4.1.3
chmod +x configure
./configure
chmod +x bootstrap.sh
./bootstrap.sh
make
sudo make install
```

And for windows, just download the binaries in from the release-page.

Run yara and verify version: 
```shell
┌──(kryssar㉿kali)-[/mnt/hgfs/VMSHARED/tryhackme/yara]
└─$ yara -v
4.1.3
```


## Task 4 - Deploy

Deploy the VM for this room or run Yara directly on your machine.


## Task 5 - Introduction to Yara Rules

Basic function is: `yara (rule) (file / dir / process ID)`  

Every rule needs a name and a condition.  

Example for running a rule on a directory: `yara myrule.yar directory`  

Example rule: 
```JSON 
rule examplerule {
    condition: true
}
```

run the test rule: 
```shell
┌──(kryssar㉿kali)-[/mnt/hgfs/VMSHARED/tryhackme/yara]
└─$ yara myrule.yar test 
examplerule test
```


## Task 6 - Expanding on Yara Rules

More information about how to write rules: [yara docs](https://yara.readthedocs.io/en/stable/writingrules.html)

Another good infographic: [Anatomy of a Yara rule](https://medium.com/malware-buddy/security-infographics-9c4d3bd891ef#18dd)


## Task 7 - Yara Modules

Frameworks such as the [Cuckoo Sandbox](https://cuckoosandbox.org/) or [Python's PE Module](https://pypi.org/project/pefile/) allows you to improve the technicality of your Yara rules ten-fold.

Sounds like nice things to keep track of, especially the sandbox because it can create Yara rules from the behaviour of the file in a sandbox environment.


## Task 8 - Other tools and Yara 

Instead of always writing rules from scratch, there's a good github [repo](https://github.com/InQuest/awesome-yara) to use as a base, and improve/build upon that. 

[LOKI](https://github.com/Neo23x0/Loki/releases) , open source IOC scanner  
[THOR](https://www.nextron-systems.com/thor-lite/) , Open source IOC and Yara scanner.
[FENRIR](https://github.com/Neo23x0/Fenrir) , Bash IOC checker 
[YAYA](https://github.com/EFForg/yaya) , Yet Another Yara Automaton


## Task 9 - Using LOKI and its Yara rule set 

Install LOKI or run it from the attached VM, I chose to run it on the windows host: 
```shell
https://github.com/Neo23x0/Loki/releases/download/v0.44.2/loki_0.44.2.zip
unzip loki_0.44.2.zip
```

then run `loki.exe` (or don't , because it starts scanning C:\ right away after it's updated definitions and rules.)

lets try to install it on kali as well to see how it works: 
```shell
https://github.com/Neo23x0/Loki/archive/refs/tags/v0.44.2.tar.gz

tar -xzvf v0.44.2.tar.gz

cd Loki-0.44.2

sudo apt install -y python3-pip

sudo pip3 install -r requirements.txt

sudo python3 loki-upgrader.py
````

Once all of that is done, run loki with: `sudo python3 loki.py --help` to verify that everything works.


To answer the questions on THM we need to run the VM that is included with the room, so lets fire that up and see how LOKI behaves. 

Scan the file under `suspicious-files` , does Loki detect this file as suspicious/malicious or benign ?  
answer:  `suspicious`

```shell
cmnatic@thm-yara:~/suspicious-files$ python ../tools/Loki/loki.py -p .

[WARNING] 
FILE: ./file1/ind3x.php SCORE: 70 TYPE: PHP SIZE: 80992 
FIRST_BYTES: 3c3f7068700a2f2a0a09623337346b20322e320a / <?php/*b374k 2.2 
MD5: 1606bdac2cb613bf0b8a22690364fbc5 
SHA1: 9383ed4ee7df17193f7a034c3190ecabc9000f9f 
SHA256: 5479f8cd1375364770df36e5a18262480a8f9d311e8eedb2c2390ecb233852ad CREATED: M
on Nov  9 15:15:32 2020 MODIFIED: Mon Nov  9 13:06:56 2020 ACCESSED: Wed Dec 22 12:
10:03 2021 
REASON_1: Yara Rule MATCH: webshell_metaslsoft SUBSCORE: 70 
DESCRIPTION: Web Shell - file metaslsoft.php REF: - 
MATCHES: Str1: $buff .= "<tr><td><a href=\\"?d=".$pwd."\\">[ $folder ]</a></td><td>
LINK</t
```

What Yara rule did it match on? : `webshell_metaslsoft`

What does Loki classify this file as?: `Web Shell`

Based on the output, what string within the Yara rule did it match on?: `Str1`

What is the name and version of this hack tool?: `b374k 2.2`

Inspect the actual Yara file that flagged file 1. Within this rule, how many strings are there to flag this file?: `1`

```shell
vim /home/cmnatic/tools/Loki/signature-base/yara/thor-webshells.yar

rule webshell_metaslsoft {
meta:
description = "Web Shell - file metaslsoft.php"
license = "https://creativecommons.org/licenses/by-nc/4.0/"
author = "Florian Roth"
date = "2014/01/28"
score = 70
hash = "aa328ed1476f4a10c0bcc2dde4461789"
strings:
$s7 = "$buff .= \"<tr><td><a href=\\\"?d=\".$pwd.\"\\\">[ $folder ]
</a></td><td>LINK</t"
condition:
all of them
```

Scan file 2. Does Loki detect this file as suspicious/malicious or benign?

```shell
loki.py -p  ~/suspicious-files/file2/index.php
```

the file seems to be clean, answer: `benign`

Inspect file 2. What is the name and version of this web shell?
answer: `b374k 3.2.3`

```shell
cmnatic@thm-yara:~/suspicious-files/file2$ head -3 1ndex.php 
<?php
/*
b374k shell 3.2.3
```

## Task 10 - Creating Yara rules with yarGen

Because yara didn't flag the `1ndex.php` file, a rule needs to be created in order to see whether it's present on any other servers.

might not be a bad idea to install [yarGen](https://github.com/Neo23x0/yarGen) locally on the kali machine as well. For now, lets continue with running it on the VM for this room. 

do remember to run: `python3 yarGen.py --update` if running locally to get the latest definition files. yarGen contains good-opcodes in order to exclude those when generating Yara rules from malicious files.

generate a rule for the `file2` under `suspicious-files`

```shell
cmnatic@thm-yara:~/tools/yarGen$ python3 yarGen.py -m /home/cmnatic/suspicious-files/file2 --excludegood -o /home/cmnatic/suspicious-files/file2.yar
```

after a while the rule is created and we can use it to see if the `1ndex.php` file is flagged this time. 

Another good tool to read up on is [yarAnalyzer](https://github.com/Neo23x0/yarAnalyzer/)

recommended reading for creating Yara rules: 
-   [write-simple-sound-yara-rules/](https://www.bsk-consulting.de/2015/02/16/write-simple-sound-yara-rules/)  
-   [how-to-write-simple-but-sound-yara-rules-part-2/](https://www.bsk-consulting.de/2015/10/17/how-to-write-simple-but-sound-yara-rules-part-2/)
-   [how-to-write-simple-but-sound-yara-rules-part-3/](https://www.bsk-consulting.de/2016/04/15/how-to-write-simple-but-sound-yara-rules-part-3/)

from within the root of the suspicious files directory, what command would you run to test Yara and your Yara rule against file 2?

```shell
cmnatic@thm-yara:~/suspicious-files$ yara file2.yar file2/1ndex.php
```

Did yara flag rule flag file 2? : `Yay`

Copy the Yara rule you created into the Loki signatures directory.
```shell
cmnatic@thm-yara:~/suspicious-files$ cp file2.yar ../tools/Loki/signature-base/yara/
```

Test the Yara rule with Loki, does it flag file 2? : `Yay`

```shell
cmnatic@thm-yara:~/tools/Loki$ python loki.py -p ~/suspicious-files/file2/

[WARNING] 
FILE: /home/cmnatic/suspicious-files/file2/1ndex.php SCORE: 70 TYPE: PHP SIZE: 2239
78 
FIRST_BYTES: 3c3f7068700a2f2a0a09623337346b207368656c / <?php/*b374k shel 
MD5: c6a7ebafdbe239d65248e2b69b670157 
SHA1: 3926ab64dcf04e87024011cf39902beac32711da 
SHA256: 53fe44b4753874f079a936325d1fdc9b1691956a29c3aaf8643cdbd49f5984bf CREATED: M
on Nov  9 15:16:03 2020 MODIFIED: Mon Nov  9 13:09:18 2020 ACCESSED: Wed Dec 22 12:
10:03 2021 
REASON_1: Yara Rule MATCH: _home_cmnatic_suspicious_files_file2_1ndex SUBSCORE: 70 
DESCRIPTION: file2 - file 1ndex.php REF: https://github.com/Neo23x0/yarGen 
MATCHES: Str1: var Zepto=function(){function G(a){return a==null?String(a):z[A.call
(a)]||"object"}function H(a){return G(a)=="function"}fun Str2: $c ... (truncated)
[NOTICE] Results: 0 alerts, 1 warnings, 7 notices
[RESULT] Suspicious objects detected!
[RESULT] Loki recommends a deeper analysis of the suspicious objects.
```

What is the name of the variable for the string that it matched on? : `Zepto`

Inspect the Yara rule, how many strings were generated? : `20`

One of the conditions to match on the Yara rule specifies file size. The file has to be less than what amount? : `700KB`


## Task 11 - Valhalla

[Valhalla](https://www.nextron-systems.com/valhalla/) is yet another feature from `Florian Roth` , this person has contributed an insane amount of time and energy providing these tools and rules to the community. 

link to valhalla-tool: [valhalla-tool](https://valhalla.nextron-systems.com/)

Enter the SHA256 hash of file 1 into Valhalla. Is this file attributed to an APT group? : `Yay`

Do the same for file 2. What is the name of the first Yara rule to detect file 2? : `Webshell_b374k_rule1`

Examine the information for file 2 from Virus Total (VT). The Yara Signature Match is from what scanner? : `THOR apt scanner`

Enter the SHA256 hash of file 2 into VT, did every AV detect this as malicious? : `Nay`

Besides .PHP , what other extension is recorded for this file? : `exe`

What JavaScript library is used by file 2? 
answer: Analyze the [github](https://github.com/b374k/b374k) repo, and see what libraries that are being used/included. 
Answer: `Zepto`

```php
/* JAVASCRIPT AND CSS FILES START */

$zepto_code = packer_read_file($GLOBALS['packer']['base_dir']."zepto.js");
```

Is this Yara rule in the default Yara file Loki uses to detect these types of hack tools? : `Nay`

## Task 12 - Conclusion

Read the text , grab a coffee and feel good about completing yet another room on THM.



EOF