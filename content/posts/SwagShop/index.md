+++
title = "HTB - SwagShop"
date = 2024-11-11
+++

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

Feroxbuster shows an `index.php/admin` which peaks my interest.
Credentials in `/app/etc/local.mxl`. Can use `magescan` to validate and find as well.
Searchsploit `37977.py`. Delete comments, change credentials (`youre:cooked`), and update target path (`index.php/admin`)
Go to the panel, then Catalog > Manage Products >HTB Sticker > Custom Options > Fill out field to be receptive to PHP file
Use PHP shell to upload once configured. You can upload on the sticker item on the homepage.
Once uploaded to the cart, visit `http://swagshop.htb/media/custom_options/quote/s/h/` to find shell!
User:`1da99f93df43ba2fd0ad651b9c1df3d4`

### Privilege Escalation
`sudo -l`, we can take advantage of the `vi` binary. Reference GTFOBins
`sudo /usr/bin/vi /var/www/html/Oof` then `:!/bin/bash`
Root:`64761fc48868d981c7894193bc54d677`
