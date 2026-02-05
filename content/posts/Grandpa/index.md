+++
title = "HTB - Grandpa"
date = 2024-09-07
+++

#msfconsole 
### Reconnaissance
```
┌──(root㉿kali)-[/home/li0t]
└─# nmap -sV -sC --open -p- 10.10.10.14
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-07 15:32 EDT
Nmap scan report for 10.10.10.14
Host is up (0.12s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Server Date: Sat, 07 Sep 2024 19:35:30 GMT
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   Server Type: Microsoft-IIS/6.0
|_  Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|_http-title: Under Construction
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 207.97 seconds
```
### Enumeration
- Same IIS as granny 
- msf scstoragepath from url instead of asp upload
- <img alt="Image" async src="images/Screenshot 2024-09-07 at 3.44.15 PM.png" width="800px"></img>
- migration to 32 bit
```
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System
 148   1072  cidaemon.exe
 272   4     smss.exe
 320   272   csrss.exe
 344   272   winlogon.exe
 392   344   services.exe
 404   344   lsass.exe
 580   392   svchost.exe
 668   392   svchost.exe
 732   392   svchost.exe
 772   392   svchost.exe
 788   392   svchost.exe
 924   392   spoolsv.exe
 952   392   msdtc.exe
 1072  392   cisvc.exe
 1112  392   svchost.exe
 1168  392   inetinfo.exe
 1204  392   svchost.exe
 1316  392   VGAuthService.exe
 1392  392   vmtoolsd.exe
 1504  392   svchost.exe
 1600  392   svchost.exe
 1768  344   logon.scr
 1804  392   dllhost.exe
 1896  392   alg.exe
 1904  580   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 2404  580   wmiprvse.exe
 2572  3408  rundll32.exe       x86   0                                      C:\WINDOWS\system32\rundll32.exe
 2768  788   HelpSvc.exe
 3408  1504  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 3480  580   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 4028  1072  cidaemon.exe
 4072  1072  cidaemon.exe

meterpreter > migrate 3480
[*] Migrating from 2572 to 3480...
[*] Migration completed successfully.

```
sysinfo<img alt="Image" async src="images/Screenshot 2024-09-07 at 3.46.28 PM.png" width="800px"></img>
local exploit suggester
- <img alt="Image" async src="images/Screenshot 2024-09-07 at 3.51.10 PM.png" width="800px"></img>
- param and privesc<img alt="Image" async src="images/Screenshot 2024-09-07 at 3.53.07 PM.png" width="800px"></img>

user: bdff5ec67c3cff017f2bedc146a5d869

root: 9359e905a2c35f861f6a57cecf28bb7b

### Exploit
### User
### Flag