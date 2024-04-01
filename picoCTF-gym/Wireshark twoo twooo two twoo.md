

pcapng file from challenge , need to find suspicious traffic , after some analysis it turns out that it was DNS smuggling. 

lots of fake flags and red herrings 

```shell


$ tshark -r shark2.pcapng -Y 'dns.qry.type == 1 && !(dns.flags == 0x8183) && dns.qry.name contains local && ip.dst==18.217.1.57'                                                                                  1 ⨯
 1637   9.440363 192.168.38.104 → 18.217.1.57  DNS 109 Standard query 0x1dd2 A cGljb0NU.reddshrimpandherring.com.windomain.local
 2046  11.972605 192.168.38.104 → 18.217.1.57  DNS 109 Standard query 0xabb9 A RntkbnNf.reddshrimpandherring.com.windomain.local
 2448  14.605726 192.168.38.104 → 18.217.1.57  DNS 109 Standard query 0x9e21 A M3hmMWxf.reddshrimpandherring.com.windomain.local
 3153  16.506492 192.168.38.104 → 18.217.1.57  DNS 109 Standard query 0x2ee1 A ZnR3X2Rl.reddshrimpandherring.com.windomain.local
 3442  18.340155 192.168.38.104 → 18.217.1.57  DNS 109 Standard query 0x2a4b A YWRiZWVm.reddshrimpandherring.com.windomain.local
 3982  20.369626 192.168.38.104 → 18.217.1.57  DNS 105 Standard query 0x4068 A fQ==.reddshrimpandherring.com.windomain.local
 4374  22.583745 192.168.38.104 → 18.217.1.57  DNS 105 Standard query 0x7418 A fQ==.reddshrimpandherring.com.windomain.local

```


save the domains: 

```shell 
tshark -r shark2.pcapng -Y 'dns.qry.type == 1 && !(dns.flags == 0x8183) && dns.qry.name contains local && ip.dst==18.217.1.57' > domains.txt
```

sort out the unique ones and only grab the first part of the domain requests 

```
cGljb0NU
RntkbnNf
M3hmMWxf
ZnR3X2Rl
YWRiZWVm
fQ==
```

it's starting to look a lot like base64 

```shell
$ echo -n "cGljb0NU                                                              
RntkbnNf
M3hmMWxf
ZnR3X2Rl
YWRiZWVm
fQ==" | base64 -d
picoCTF{dns_3xf1l_ftw_deadbeef}
```

flag: picoCTF{dns_3xf1l_ftw_deadbeef}



