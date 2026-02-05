+++
title = "HTB - Lame"
date = 2024-09-16
+++

<img alt="Image" async src="images/Lame.png" width="800px"></img>

Lame is a very simple and easy box detailing the importance of SMB enumeration, and how to capitalize on said enumeration with metasploit. Let's get to it.
### Scan
Scan: `nmap -sV -sC --open -p- 10.10.10.3`
Result:
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-02 22:06 EDT
Nmap scan report for 10.10.10.3
Host is up (0.13s latency).
Not shown: 65530 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 2h00m21s, deviation: 2h49m45s, median: 19s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2024-09-02T22:10:17-04:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 239.59 seconds
```

There are a good amount of options here for enumeration. We've got FTP via vsftpd 2.3.4, we've got OpenSSH 4.7p1, amd some SMB shares. Ideally, we'd go through each, but SMB is the answer here, and we'll keep it concise.

### Initial Access

Researching this version of Samba 3.0.20, we come across a CVE-2007-2447, which allows remote attackers to execute arbitrary commands via shell meta-characters involving the `SamrChangePassword` function, when the "username map script" smb.conf option is enabled. We can search metasploit modules for an answer and come up with one:

`use exploit/multi/samba/usermap_script`

<img alt="Image" async src="images/Screenshot 2024-09-02 at 10.38.35 PM.png" width="800px"></img>

Looking at our options, it's imperative to set LHOST as our tun0 address and not our eth0. This threw many errors and is worth consideration in the future. 

Parameters, otherwise, are set as normal.

<img alt="Image" async src="images/Screenshot 2024-09-02 at 10.40.01 PM.png" width="800px"></img>

### Post Exploitation

We can do what we've done before, and cd straight to home directory for our user flag. Then, we can navigate to the root folder for our root flag.

`USER:1d4a90b8ef1da1772cad2ef968d96bad`

`ROOT:e82a028aebebb3f5c28034f9561ba802`

#easy #windows #msfconsole 