+++
title = "HTB - Active"
date = 2025-06-22
+++

<img alt="Image" async src="images/Active.png" width="800px"></img>

Active is a machine to learn about the enumeration of services in an AD environment. Attackers will need to leverage understandings of ports and services, SMB configurations, Group Policy Preference (GPP), hash cracking, and kerberoasting. Without further ado, let's get into it.

### Scan
```
└─# nmap -sV -sC --open -p- 10.10.10.100
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-10 20:53 EDT
Nmap scan report for 10.10.10.100
Host is up (0.14s latency).
Not shown: 65513 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft sDNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-09-11 00:54:13Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
49166/tcp open  msrpc         Microsoft Windows RPC
49168/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-09-11T00:55:13
|_  start_date: 2024-09-11T00:51:08
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 135.17 seconds

```

Our TCP and UDP port scan didn't finish before we had achieved root, so I'll leave those scans out. Do remember, it is extremely important to cover all bases in the enumeration phase of our reconnaissance and scanning. 

We've got quite a lot going on here, and I'd like to go through these ports and their importance within the context of Active Directory. `88` is the one that catches my eye, because it is occupied by the `Kerberos` service which is used to authenticate Users to the Domain Controller. `139` and `445` are associated with the `SMB` (Server Messaging Block) service, and we can usually enumerate these services for information of files.`389` and `3268` are for the `LDAP` services, and `MSRPC` allows the hosts to talk to each other. 

### Enumeration

We've got a Windows 2008 Server acting as our Domain Controller. We get the name of the machine too, so add it to `/etc/hosts` The normal vector here is to use the DNS Resolver to get our hostname `active.htb` and try to exploit through a web-based attack vector. We can use commands such as like `host -l`, `dig`, and `nslookup`. These are dead-ends, though, and we need to start moving on to other options.

In order to interact with SMB shares, we can use tools like `smbclient` and `smbmap`. The latter allows us to enumerate the SMB shares to get information about files, share, and directories. The former allows us to interact with and connect to actual shares as well as transfer files. 

Using the scan `smbmap -H active.htb`, we get access to see some shares:

<img alt="Image" async src="images/Screenshot 2024-09-10 at 9.49.47 PM.png" width="800px"></img>

We've got access to only one share, so lets enumerate a bit more. We can connect via SMB to the machine using the syntax below, and the `-N` parameter suppresses the password because we're trying for an anonymous logon.

`smbclient //active.htb/Replication -N`

### Initial Access

Once we're connected, we can browse around using `cd` and whatnot, while `get` and `more` give us information about files. Just like when fuzzing web directories, there are key items and things to look for in shares as well. Whenever a new Group Policy Preference (GPP) is created, there’s an xml file created in the SYSVOL share with that config data, including any passwords associated with the GPP. For security, Microsoft AES encrypts the password before it’s stored as `cpassword`. These are extremely helpful when looking for initial footholds.

<img alt="Image" async src="images/Screenshot 2024-09-10 at 10.06.08 PM.png" width="800px"></img>

When we open the file with `more`, or transfer it with `get`, we'll see that there are credentials here.

<img alt="Image" async src="images/Screenshot 2024-09-10 at 10.08.42 PM.png" width="800px"></img>

Because we know the encryption method, and the key's been published before in a MSFT leak, we've got tools to decrypt. One of them is called `gpp-decrypt`.

`gpp-decrypt [ENCRYPTED PASS]`

When we do this, we're able to get our password.

 `GPPstillStandingStrong2k18`

An important concept when enumerating on Active Directory machines is to spray credentials whenever and wherever you can, as they're often reused. We can now go use `smbclient` to log in with credentials.

`smbclient -W active.htb -U SVC_TGS //active.htb/USERS`

Once in, we can navigate to the user's directory and desktop for our flag.

<img alt="Image" async src="images/Screenshot 2024-09-10 at 10.20.51 PM.png" width="800px"></img>

`USER:77d587ba5436a9188f242c777ce6fd17`

### Privilege Escalation

A very useful and common vector for escalating privileges in this environment is by abusing the AD authentication service known as Kerberos. The diagram below illustrates how the protocol works.

<img alt="Image" async src="images/Screenshot 2024-09-11 at 9.27.35 AM.png" width="800px"></img>

If you compromise a user that has a valid kerberos ticket-granting ticket (TGT), then you can request one or more ticket-granting service (TGS) service tickets for any Service Principal Name (SPN) from a domain controller. An example SPN would be the Application Server shown in the above figure.

A portion of the TGS ticket is encrypted with the hash of the service account associated with the SPN. Therefore, you can run an offline brute force attack on the encrypted portion to reveal the service account password. Therefore, if you request an administrator account TGS ticket and the administrator is using a weak password, we’ll be able to crack it!

A tool called `impacket` is vital for us in this process, and it can be installed with a `git` repository, and installed with `python setup.py install` once in the downloaded directory. `GetUserSPNs.py` is a great script that enumerates SPNs for us, and the command to use it is below. (It sit in `/examples/` directory)

`./GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request`

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.02.19 PM.png" width="800px"></img>

We get the TGS for users in a useful JtR/Hashcat format that we can use to brute-force offline. It's important to export these results to a file for ease of use, and additionally, if the hash or list is going to create a computationally intensive process, run the process on metal. VM is no good here. 

If you receive an error, “Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)”, it’s probably because the attack machine date and time are not in sync with the Kerberos server.

We can take our TGS and use any cracking tool like `john` or `hashcat` and crack!

`john --wordlist=/usr/share/wordlists/rockyou.txt spn-admin.txt`

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.13.12 PM.png" width="800px"></img>

We got a password! 

Now, remember what I said. We've got a login, enumerate once again. `smbmap` is great for this to re-assess where we are.

`smbmap -H 10.10.10.100 -d active.htb -u administrator -p Ticketmaster1968`

We're looking for access to any important drive or directory. In this case, we've got access to $C drive! All we need to do is connect via `smbclient` with our credentials, and grab our flag.

`smbclient //10.10.10.100/C$ -U active.htb\\administrator%Ticketmaster1968`

`more \users\administrator\desktop\root.txt`

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.18.00 PM.png" width="800px"></img>

`ROOT:3dfddeb6977b22a2d47e8dc8346cbda1`

#windows #manual #easy 
