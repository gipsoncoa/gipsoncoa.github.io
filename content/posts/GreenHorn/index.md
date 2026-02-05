+++
title = "HTB - GreenHorn"
date = 2024-10-08
+++

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
SF:-gnu%r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-T
SF:ype:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400
SF:\x20Bad\x20Request")%r(GetRequest,2A60,"HTTP/1\.0\x20200\x20OK\r\nCache
SF:-Control:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform
SF:\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nSet-Cookie:\x20i_li
SF:ke_gitea=1278cac7ce6b83ed;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nS
SF:et-Cookie:\x20_csrf=EmpPa7lhWkY1m0Ejzd1STHwTCMA6MTcyODQzMjI1MTcyMTk2ODM
SF:0Ng;\x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Fra
SF:me-Options:\x20SAMEORIGIN\r\nDate:\x20Wed,\x2009\x20Oct\x202024\x2000:0
SF:4:11\x20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class
SF:=\"theme-auto\">\n<head>\n\t<meta\x20name=\"viewport\"\x20content=\"wid
SF:th=device-width,\x20initial-scale=1\">\n\t<title>GreenHorn</title>\n\t<
SF:link\x20rel=\"manifest\"\x20href=\"data:application/json;base64,eyJuYW1
SF:lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Im
SF:h0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9nc
SF:mVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9w
SF:bmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjM
SF:wMDAvYX")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type
SF::\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x2
SF:0Bad\x20Request")%r(HTTPOptions,1A4,"HTTP/1\.0\x20405\x20Method\x20Not\
SF:x20Allowed\r\nAllow:\x20HEAD\r\nAllow:\x20HEAD\r\nAllow:\x20GET\r\nCach
SF:e-Control:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transfor
SF:m\r\nSet-Cookie:\x20i_like_gitea=7168d0969ce618ef;\x20Path=/;\x20HttpOn
SF:ly;\x20SameSite=Lax\r\nSet-Cookie:\x20_csrf=CL-hqo3tMizdc2__mQEw7jm9LGw
SF:6MTcyODQzMjI1NzM5NjI4OTQ0OA;\x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\
SF:x20SameSite=Lax\r\nX-Frame-Options:\x20SAMEORIGIN\r\nDate:\x20Wed,\x200
SF:9\x20Oct\x202024\x2000:04:17\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r
SF:(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x2
SF:0text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad
SF:\x20Request");
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
password<img alt="Image" async src="images/Screenshot 2024-10-08 at 10.28.35 PM.png" width="800px"></img> `sidefromsidetheothersidesidefromsidetheotherside`
ssh into box
FINALLY ROOT FLAG FYCK
Root:0a6a92e75ef9baf2da20d2cd8a8e1e68
