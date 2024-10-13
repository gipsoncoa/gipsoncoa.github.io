### Scan
```
┌──(root㉿kali)-[/home/li0t/Desktop/HTB/Editorial]
└─# nmap -sVC -p- --open 10.10.11.20
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-05 09:35 EDT
Nmap scan report for editorial.htb (10.10.11.20)
Host is up (0.14s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)
|_  256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Editorial Tiempo Arriba
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.83 seconds
```
### Enum
Ferox ![[Screenshot 2024-10-05 at 10.31.50 AM.png]]

tiempo arriba domain![[Screenshot 2024-09-13 at 6.31.36 PM.png]]
