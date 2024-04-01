# Writeup for the XXXX rated HTB box YYYY

![abcd](abcd.png)


## Recon

## Scanning

### Port scanning with NMAP
```shell
sudo nmap -T4 -Pn -sV -A XXYY.htb -o nmap.init.txt
```

### Directory scanning with DIRB
```shell
sudo dirb http://XXYY.htb -o dirb.txt
```

### Vulnerability scanning with NIKTO
```shell
sudo nikto -h http://XXYY.htb -o nikto.html
```

### Scanning for subdomains with FFUF
```shell
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://secret.htb/ | tee ffuf.txt
```

## Gaining Access
### Lateral movement
### Privledge escalation to root / Administrator

<!-- 
## Maintaining Access
## Clearing Track
-->

## Summary
