---
title: SwagShop - Hack the Box
date: 2024-10-13 12:29:12 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/SwagShop.png
---
---
title: SwagShop_(I) - Hack the Box
date: 2024-10-13 12:07:03 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/SwagShop_(I).png
---
---
title: 2024-10-12-SwagShop_(I) - Hack the Box
date: 2024-10-13 12:01:04 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/2024-10-12-SwagShop_(I).png
---
#unfinished #linux #manual 
### Scan
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-11 15:44 EDT
Nmap scan report for 10.10.10.140
Host is up (0.15s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
|_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Did not follow redirect to http://swagshop.htb/
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.78 seconds
```
### Enumeration

ferox![Image](/assets/Screenshot 2024-09-11 at 5.29.10 PM.png)
admin panel![Image](/assets/Screenshot 2024-09-11 at 5.27.01 PM.png)
local xml![Image](/assets/Screenshot 2024-09-11 at 5.09.24 PM.png)
predates version2![Image](/assets/Screenshot 2024-09-11 at 5.13.42 PM.png)
pyscript![Image](/assets/Screenshot 2024-09-11 at 5.25.28 PM.png)
No auth, keep going
try original searchsploit (37977.py)
doctor script to fit needs, really interesting exploit to break down in here wahts going on. forme and forme and all
admin panel![Image](/assets/Screenshot 2024-09-11 at 5.47.56 PM.png)
Go back to original rce script now that youve got creds (forme forme)
RCE ![Image](/assets/Screenshot 2024-09-11 at 5.54.12 PM.png)



### Initial Access
### Privilege Escalation
###
