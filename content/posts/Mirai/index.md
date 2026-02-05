+++
title = "HTB - Mirai"
date = 2024-09-07
+++

### Reconnaissance
```
└─# nmap -sV -sC --open -p- 10.10.10.48
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-07 21:58 EDT
Nmap scan report for 10.10.10.48
Host is up (0.12s latency).
Not shown: 65515 closed tcp ports (reset), 14 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp    open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp    open  http    lighttpd 1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: lighttpd/1.4.35
1058/tcp  open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
32400/tcp open  http    Plex Media Server httpd
|_http-favicon: Plex
|_http-cors: HEAD GET POST PUT DELETE OPTIONS
|_http-title: Unauthorized
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
32469/tcp open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 63.66 seconds

```

###
- landing page<img alt="Image" async src="images/Screenshot 2024-09-07 at 10.18.03 PM.png" width="800px"></img>
- gobuster w need for cred<img alt="Image" async src="images/Screenshot 2024-09-07 at 10.18.49 PM.png" width="800px"></img>
- same <img alt="Image" async src="images/Screenshot 2024-09-07 at 10.19.06 PM.png" width="800px"></img>
- rce for auth user<img alt="Image" async src="images/Screenshot 2024-09-07 at 10.19.24 PM.png" width="800px"></img>
- default creds (not for portal, but worked with ssh)<img alt="Image" async src="images/Screenshot 2024-09-07 at 10.20.01 PM.png" width="800px"></img>
- user : ff837707441b257a20e32199d7c8838d
- privesc and flag prank<img alt="Image" async src="images/Screenshot 2024-09-07 at 10.21.40 PM.png" width="800px"></img>
- Pranked agan <img alt="Image" async src="images/Screenshot 2024-09-07 at 10.23.14 PM.png" width="800px"></img>
- Strings command (very cool) Chat GPGOAT<img alt="Image" async src="images/Screenshot 2024-09-07 at 10.40.03 PM.png" width="800px"></img>
- <img alt="Image" async src="images/Screenshot 2024-09-07 at 10.37.43 PM.png" width="800px"></img>