+++
title = "HTB - PermX"
date = 2024-09-10
+++

#linux #manual 
### Scan
```
└─# nmap -sV -sC --open -p- 10.10.11.23
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-10 13:40 EDT
Nmap scan report for 10.10.11.23
Host is up (0.13s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e2:5c:5d:8c:47:3e:d8:72:f7:b4:80:03:49:86:6d:ef (ECDSA)
|_  256 1f:41:02:8e:6b:17:18:9c:a0:ac:54:23:e9:71:30:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://permx.htb
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.68 seconds

```

### Enumeration

Landing Page  <img alt="Image" async src="images/Screenshot 2024-09-10 at 1.43.08 PM.png" width="800px"></img>

MF IMPORTANTCE OF WRDLIST AND RECON <img alt="Image" async src="images/Screenshot 2024-09-10 at 12.17.42 PM 1.png" width="800px"></img>

We try to brute with hydra but no dice (still include)

We look up version info and find git repo of RCE (Include dets) cve-2023-4220

Get revshell

stabilize

serv linpeas
serv linenum 
config files, users, versions

sudo abuse<img alt="Image" async src="images/Screenshot 2024-09-10 at 12.17.42 PM 2.png" width="800px"></img> dead end

 cred<img alt="Image" async src="images/Screenshot 2024-09-10 at 6.34.09 PM.png" width="800px"></img>
 
lateral movement <img alt="Image" async src="images/Screenshot 2024-09-10 at 6.37.56 PM.png" width="800px"></img>

`USER:b8fd18fc6b2e99f43d1dc34f364cde77`

linepeas... again

sudo -l. boy we're cooking<img alt="Image" async src="images/Screenshot 2024-09-10 at 6.44.41 PM.png" width="800px"></img>

actually, we're not...

symbolic link

<img alt="Image" async src="images/Screenshot 2024-09-10 at 7.16.48 PM.png" width="800px"></img>

honestly no idea why the fucl that worked. read up on it. 

`ROOT:0606092a36421c32e83576773090f5d2`