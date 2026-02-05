+++
title = "HTB - Boardlight"
date = 2024-09-08
+++

### Reconnaissance

```
┌──(root㉿kali)-[/home/li0t]
└─# nmap -sV -sC --open -p- 10.10.11.11 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-08 07:13 EDT
Nmap scan report for 10.10.11.11
Host is up (0.13s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.91 seconds

```

###
- Wordpress sight (maybe)
- site version <img alt="Image" async src="images/Screenshot 2024-09-08 at 7.17.32 AM.png" width="800px"></img>
- fuzzing (go into det about how got here)<img alt="Image" async src="images/Screenshot 2024-09-08 at 7.57.57 AM.png" width="800px"></img>
- <img alt="Image" async src="images/Screenshot 2024-09-08 at 7.59.18 AM.png" width="800px"></img>
- Dolibarr 17.0.0 CVE<img alt="Image" async src="images/Screenshot 2024-09-08 at 8.01.15 AM.png" width="800px"></img>
- looks like PHP code injection
- info<img alt="Image" async src="images/Screenshot 2024-09-08 at 8.06.21 AM.png" width="800px"></img>
- github python remote shell
- shell connect and stabilize (wlrap too)
- creds in conf file<img alt="Image" async src="images/Screenshot 2024-09-08 at 8.58.48 AM.png" width="800px"></img>
- <img alt="Image" async src="images/Screenshot 2024-09-08 at 8.46.47 AM.png" width="800px"></img>
- umm completely hidden home dir with user<img alt="Image" async src="images/Screenshot 2024-09-08 at 8.57.37 AM.png" width="800px"></img>
- larissa ssh w creds <img alt="Image" async src="images/Screenshot 2024-09-08 at 9.00.32 AM.png" width="800px"></img>
- cat user : 93bc590302d174c7046be6b70f8ce1b5
- py server to host exploit<img alt="Image" async src="images/Screenshot 2024-09-08 at 9.17.08 AM.png" width="800px"></img>
- make sure in home dir and lead compromised machine to it<img alt="Image" async src="images/Screenshot 2024-09-08 at 9.16.30 AM.png" width="800px"></img>
- change perms with chmod 777
- run with ./exploit.sh
- cat root: 9292487b40387fbc01785055950f3076
