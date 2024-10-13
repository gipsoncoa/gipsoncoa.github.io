---
title: GreenHorn - Hack the Box
date: 2024-10-13 12:29:12 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/GreenHorn.png
---
---
title: GreenHorn - Hack the Box
date: 2024-10-13 12:07:03 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/GreenHorn.png
---
---
title: 2024-10-12-GreenHorn - Hack the Box
date: 2024-10-13 12:01:04 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/2024-10-12-GreenHorn.png
---
### Scan
```
└─$ nmap -sV -sC -p- --open 10.10.11.25
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-08 20:03 EDT
Nmap scan report for 10.10.11.25
Host is up (0.12s latency).
Not shown: 53409 closed tcp ports (conn-refused), 12123 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 57:d6:92:8a:72:44:84:17:29:eb:5c:c9:63:6a:fe:fd (ECDSA)
|_  256 40:ea:17:b1:b6:c5:3f:42:56:67:4a:3c:ee:75:23:2f (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://greenhorn.htb/
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=1278cac7ce6b83ed; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=EmpPa7lhWkY1m0Ejzd1STHwTCMA6MTcyODQzMjI1MTcyMTk2ODM0Ng; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Wed, 09 Oct 2024 00:04:11 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>GreenHorn</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYX
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=7168d0969ce618ef; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=CL-hqo3tMizdc2__mQEw7jm9LGw6MTcyODQzMjI1NzM5NjI4OTQ0OA; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Wed, 09 Oct 2024 00:04:17 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.94SVN%I=7%D=10/8%Time=6705C87B%P=aarch64-unknown-linux
SF:-gnu%r(GenericLines,67,"HTTP/1\.1 400 Bad Request
Content-T
SF:ype: text/plain; charset=utf-8
Connection: close

400
SF: Bad Request")%r(GetRequest,2A60,"HTTP/1\.0 200 OK
Cache
SF:-Control: max-age=0, private, must-revalidate, no-transform
SF:
Content-Type: text/html; charset=utf-8
Set-Cookie: i_li
SF:ke_gitea=1278cac7ce6b83ed; Path=/; HttpOnly; SameSite=Lax
S
SF:et-Cookie: _csrf=EmpPa7lhWkY1m0Ejzd1STHwTCMA6MTcyODQzMjI1MTcyMTk2ODM
SF:0Ng; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
X-Fra
SF:me-Options: SAMEORIGIN
Date: Wed, 09 Oct 2024 00:0
SF:4:11 GMT

<!DOCTYPE html>
<html lang=\"en-US\" class
SF:=\"theme-auto\">
<head>
	<meta name=\"viewport\" content=\"wid
SF:th=device-width, initial-scale=1\">
	<title>GreenHorn</title>
	<
SF:link rel=\"manifest\" href=\"data:application/json;base64,eyJuYW1
SF:lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Im
SF:h0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9nc
SF:mVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9w
SF:bmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjM
SF:wMDAvYX")%r(Help,67,"HTTP/1\.1 400 Bad Request
Content-Type
SF:: text/plain; charset=utf-8
Connection: close

400
SF:0Bad Request")%r(HTTPOptions,1A4,"HTTP/1\.0 405 Method Not\
SF:x20Allowed
Allow: HEAD
Allow: HEAD
Allow: GET
Cach
SF:e-Control: max-age=0, private, must-revalidate, no-transfor
SF:m
Set-Cookie: i_like_gitea=7168d0969ce618ef; Path=/; HttpOn
SF:ly; SameSite=Lax
Set-Cookie: _csrf=CL-hqo3tMizdc2__mQEw7jm9LGw
SF:6MTcyODQzMjI1NzM5NjI4OTQ0OA; Path=/; Max-Age=86400; HttpOnly;\
SF:x20SameSite=Lax
X-Frame-Options: SAMEORIGIN
Date: Wed, 0
SF:9 Oct 2024 00:04:17 GMT
Content-Length: 0

")%r
SF:(RTSPRequest,67,"HTTP/1\.1 400 Bad Request
Content-Type:
SF:0text/plain; charset=utf-8
Connection: close

400 Bad
SF: Request");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 148.06 seconds
                                         
```
Website - ferox to find interesting php files like login and whatnot. Also pluck v 4.7.18 is vulnerable to RCE. Couldnt get it to work without authentication
no dir traversal eaither :/

`http://greenhorn.htb:3000` yields an almost git repo with site structure.
We find that passwords are sha512 encrypted, and we find a pass.php in the data/settings directory. lets crack
`iloveyou1` is password. 
Once in, we can refer back to the Poc from the exploit earlier, and install a module with our zip file containing our php reverse shell.
once its uploaded, we should get a hit.
stable ya shell
we dont have access to user, but if we `su`, we can use same password and get access
USER:
opening the openvas pdf, we see a pizelated password for root. We can use Depix to find out what it says
use pdf24 website to extract image from pdf. then unzip file
use depix in tool dir
```
python3 depix.py \
-p /home/li0t/Downloads/0.png  \ 
-s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png \
-o ooo.png

```
password![Image](/assets/Screenshot 2024-10-08 at 10.28.35 PM.png) `sidefromsidetheothersidesidefromsidetheotherside`
ssh into box
FINALLY ROOT FLAG FYCK
Root:0a6a92e75ef9baf2da20d2cd8a8e1e68
