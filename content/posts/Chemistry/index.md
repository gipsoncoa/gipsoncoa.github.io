+++
title = "HTB - Chemistry"
date = 2024-10-25
+++

```
Host is up (0.15s latency).
Not shown: 65515 closed tcp ports (reset), 18 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b6:fc:20:ae:9d:1d:45:1d:0b:ce:d9:d0:20:f2:6f:dc (RSA)
|   256 f1:ae:1c:3e:1d:ea:55:44:6c:2f:f2:56:8d:62:3c:2b (ECDSA)
|_  256 94:42:1b:78:f2:51:87:07:3e:97:26:c9:a2:5c:0a:26 (ED25519)
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.3 Python/3.9.5
|     Date: Thu, 24 Oct 2024 16:46:10 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 719
|     Vary: Cookie
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Chemistry - Home</title>
|     <link rel="stylesheet" href="/static/styles.css">
|     </head>
|     <body>
|     <div class="container">
|     class="title">Chemistry CIF Analyzer</h1>
|     <p>Welcome to the Chemistry CIF Analyzer. This tool allows you to upload a CIF (Crystallographic Information File) and analyze the structural data contained within.</p>
|     <div class="buttons">
|     <center><a href="/login" class="btn">Login</a>
|     href="/register" class="btn">Register</a></center>
|     </div>
|     </div>
|     </body>
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.94SVN%I=7%D=10/24%Time=671A79D0%P=aarch64-unknown-linu
SF:x-gnu%r(GetRequest,38A,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/3
SF:\.0\.3\x20Python/3\.9\.5\r\nDate:\x20Thu,\x2024\x20Oct\x202024\x2016:46
SF::10\x20GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-L
SF:ength:\x20719\r\nVary:\x20Cookie\r\nConnection:\x20close\r\n\r\n<!DOCTY
SF:PE\x20html>\n<html\x20lang=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20ch
SF:arset=\"UTF-8\">\n\x20\x20\x20\x20<meta\x20name=\"viewport\"\x20content
SF:=\"width=device-width,\x20initial-scale=1\.0\">\n\x20\x20\x20\x20<title
SF:>Chemistry\x20-\x20Home</title>\n\x20\x20\x20\x20<link\x20rel=\"stylesh
SF:eet\"\x20href=\"/static/styles\.css\">\n</head>\n<body>\n\x20\x20\x20\x
SF:20\n\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\n\x20\x20\x20\x20<div\x2
SF:0class=\"container\">\n\x20\x20\x20\x20\x20\x20\x20\x20<h1\x20class=\"t
SF:itle\">Chemistry\x20CIF\x20Analyzer</h1>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20<p>Welcome\x20to\x20the\x20Chemistry\x20CIF\x20Analyzer\.\x20This\x2
SF:0tool\x20allows\x20you\x20to\x20upload\x20a\x20CIF\x20\(Crystallographi
SF:c\x20Information\x20File\)\x20and\x20analyze\x20the\x20structural\x20da
SF:ta\x20contained\x20within\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<div\x
SF:20class=\"buttons\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<
SF:center><a\x20href=\"/login\"\x20class=\"btn\">Login</a>\n\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20<a\x20href=\"/register\"\x20class=\"b
SF:tn\">Register</a></center>\n\x20\x20\x20\x20\x20\x20\x20\x20</div>\n\x2
SF:0\x20\x20\x20</div>\n</body>\n<")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTML\
SF:x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x20
SF:\x20\x20\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x20
SF:\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equiv
SF:=\"Content-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x20
SF:\x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x20<
SF:/head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Err
SF:or\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\
SF:x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20reque
SF:st\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0<p>Error\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20Ba
SF:d\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20\x
SF:20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 143.72 seconds
                                                                
```

### Enumeration
- We've got a webserver running Flask. We can tell by Header! Do more research on this
- CFI File upload once we register. I wonder if we can steal the cookie like Ippsec did in the one video?
- Looks like there's a pymatgen arbitrary code execution while parsing a malicious CFI file 
- We update the malicious CFI file to include a bash one-liner, and denote the full path
- Once in, stabilize. Root around and find a user named Rosa. No access though
- In the instance directory, theres a SQLite database that we can view with sqlitebrowser
- In it, we find MD5 hashes and can decode Rosa's password. `unicorniosrosados`
- Imma try same with admin. No go
- USER: `d7f630e994eb11375cd439bff8058b01`
- Rooting around more, we find a monitoring site directory that's off limits to us. I ran a netstat to check for any hidden ports, and sure enough theres an instance running on 8080. 
- We can forward the port with SSH creds: `ssh -v -N -L 8888:localhost:8080 rosa@10.10.11.38`
- We find a website with a list of running services and graphs detailing earnings. I found a JQuery version with WAppalyzer, but no dice. Using burp to analyze the header for version info, we find `Python/3.9 aiohttp/3.9.1`
- We find CVE 2024 23334 that capitalizes on a path traversal exploit. Using a bash script on github, we try to exploit.
- We get a 404 error. Running a feroxbuster, we find that the only directory is `assets`, not `static` as the payload requires.
- Changing our payload, we can test this exploit by printing out the `/etc/passwd`
- It works! Let's cat out our root flag as well.
- ROOT:`1dad78d3ebf6768411a32ce68554904c`
- To gain access, just access the private Root ISA key for SSH purposes.