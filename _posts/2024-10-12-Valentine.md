---
title: Valentine - Hack the Box
date: 2024-10-13 12:29:12 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/Valentine.png
---
---
title: Valentine - Hack the Box
date: 2024-10-13 12:07:03 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/Valentine.png
---
---
title: 2024-10-12-Valentine - Hack the Box
date: 2024-10-13 12:01:04 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/2024-10-12-Valentine.png
---
![Image](/assets/Valentine.png)
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

![Image](/assets/Screenshot 2024-09-11 at 11.05.09 AM.png)

Entering the website, we get a pretty big hint with the landing page above - it takes a very trained eye. Heartbleed is an early SSL vulnerability that hemorrhages sensitive information through incorrect memory handling of the heartbeat expansion. We can abuse this to leak information like keys, credentials, and server system information. (Though I didn't know any of this yet.)

Using `feroxbuster`, we can uncover some directories from the server, and I also had `ffuf` in another window hunting subdomains. 

 ![Image](/assets/Screenshot 2024-09-11 at 11.07.29 AM.png)

As I've talked about before, the `/dev/` directory is almost always home to some juicy information. Here, we can extract a todo list from a developer, and a hex-coded key. We can also see and `encode.php` and `decode.php` function that was mentioned in the todo list.  We'll keep note of this for the key.

![Image](/assets/Screenshot 2024-09-11 at 11.04.40 AM.png)

![Image](/assets/Screenshot 2024-09-11 at 11.09.45 AM.png)

When we do decode the message, we're left with a Private RSA key, and I assume by the title of the file, it belongs to the user `hype`. As a disclaimer, though, be wary of decrypting sensitive files and such online, as you don't know who or what is logging them in the backend. Take the key and save it to a file `rsaprivate.txt`.

```
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
```

We can take this key to try and SSH into `hype@10.10.10.79`, but we'll be met with an error. The reason being is because the file needs correct permissions. We cannot use a behaviorally "loose" file when transferring private keys, as that'd be insecure and against the whole development of `SSH`. To change the perms to read only, we can use `chmode 400 [File]` and then try again.  `ssh -i rsaprivate.txt hype@10.10.10.79`

We're allowed an attempt to connect, but we still need a password that we don't have. We're missing something. Further enumeration is required. 

### Initial Access

We can throw things at the wall and see what sticks, so with the `feroxbuster` and `gobuster` still running, I'll use an `nmap` script for potential vulnerabilities and see what comes back.

`nmap --script vuln 10.10.10.79`

![Image](/assets/Screenshot 2024-09-11 at 11.24.04 AM.png)

There's our heartbleed confirmation. We can crawl google and find a `git` repository for this exact purpose and clone it onto our machine. 

 `git clone https://gist.github.com/10174134.git`

![Image](/assets/Screenshot 2024-09-11 at 11.27.49 AM.png)

When we run, we get some more valuable information, including an encoded string. We can go back to the exposed tool within the website and try our luck.

![Image](/assets/Screenshot 2024-09-11 at 11.28.57 AM.png) 

Believe the hype. Now with a password, we can try to login again, and use the password as the RSA_Key password. NOT the user's password (It won't work)

 ![Image](/assets/Screenshot 2024-09-11 at 11.31.31 AM.png)

The user flag resides in the directory that we've spawned in, and we can take it for our notes.

 `USER:9bb8c74e1764083f4e3f1cdd01cfefa6`

### Privilege Escalation

Next, we can server `LinPeas.sh` through our `python` server, using our favorite:

 `python -m SimpleHTTPServer 8080`

Combing through our results, a very strong opportunity for privesc lies with Tmux, as seen below.

![Image](/assets/Screenshot 2024-09-11 at 11.41.59 AM.png)

Tmux is a terminal multiplexer; it allows you to create several "pseudo terminals" from a single terminal. This is very useful for running multiple programs with a single connection, such as when you're remotely connecting to a machine using SSH. There's a session running as `root`, and we can easily join the session with `tmux -S ./dev/dev_sess`

![Image](/assets/Screenshot 2024-09-11 at 11.49.25 AM.png)

Now we've got access to our flag for our notes.

`ROOT:a498a4ce2249b580753fabae6b30f523`

Another method of privesc was through an old vulnerability called `Dirtycow`, which allows low privilege users to gain write access to otherwise read-only memory mappings and thus increase their privileges on the system. The kernel's gotta be pretty old, though.

#easy #linux #manual 
