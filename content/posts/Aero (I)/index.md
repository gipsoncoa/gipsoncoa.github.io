+++
title = "HTB - Aero (I)"
date = 2024-11-25
+++

### Scan
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-25 20:20 EST
Nmap scan report for 10.10.11.237
Host is up (0.13s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Aero Theme Hub
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 260.84 seconds
```

### Enumeration
Webpage allows for upload. ARR 3.0. Only allows for `theme` and `themepack` files.
Doing research about Windows 11 and themes, we can see "ThemeBleed"