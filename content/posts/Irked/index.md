+++
title = "HTB - Irked"
date = 2024-10-28
+++

### Scan
``` 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 13:07 EDT
Stats: 0:00:01 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 59.50% done; ETC: 13:07 (0:00:01 remaining)
Stats: 0:00:01 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 59.80% done; ETC: 13:07 (0:00:01 remaining)
Stats: 0:00:01 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 60.65% done; ETC: 13:07 (0:00:01 remaining)
Stats: 0:00:01 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 65.70% done; ETC: 13:07 (0:00:01 remaining)
Nmap scan report for 10.10.10.117
Host is up (0.14s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
111/tcp open  rpcbind

Nmap done: 1 IP address (1 host up) scanned in 2.73 seconds



---------------------Starting Nmap Basic Scan---------------------
                                                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 13:07 EDT
Nmap scan report for 10.10.10.117
Host is up (0.13s latency).

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp  open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          45010/tcp   status
|   100024  1          48072/udp6  status
|   100024  1          58879/tcp6  status
|_  100024  1          60060/udp   status
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.44 seconds



----------------------Starting Nmap UDP Scan----------------------
                                                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 13:08 EDT
Warning: 10.10.10.117 giving up on port because retransmission cap hit (1).
Nmap scan report for 10.10.10.117
Host is up (0.13s latency).
Not shown: 814 open|filtered udp ports (no-response), 184 closed udp ports (port-unreach)
PORT     STATE SERVICE
111/udp  open  rpcbind
5353/udp open  zeroconf

Nmap done: 1 IP address (1 host up) scanned in 178.40 seconds


Making a script scan on UDP ports: 111, 5353
                                                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 13:10 EDT
Nmap scan report for 10.10.10.117
Host is up (0.13s latency).

PORT     STATE SERVICE VERSION
111/udp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          45010/tcp   status
|   100024  1          48072/udp6  status
|   100024  1          58879/tcp6  status
|_  100024  1          60060/udp   status
5353/udp open  mdns    DNS-based service discovery

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.60 seconds



---------------------Starting Nmap Full Scan----------------------
                                                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 13:11 EDT
Initiating Parallel DNS resolution of 1 host. at 13:11
Completed Parallel DNS resolution of 1 host. at 13:11, 0.02s elapsed
Initiating SYN Stealth Scan at 13:11
Scanning 10.10.10.117 [65535 ports]
Discovered open port 22/tcp on 10.10.10.117
Discovered open port 111/tcp on 10.10.10.117
Discovered open port 80/tcp on 10.10.10.117
Warning: 10.10.10.117 giving up on port because retransmission cap hit (1).
Discovered open port 8067/tcp on 10.10.10.117
SYN Stealth Scan Timing: About 10.58% done; ETC: 13:15 (0:04:22 remaining)
SYN Stealth Scan Timing: About 20.26% done; ETC: 13:16 (0:04:00 remaining)
Discovered open port 65534/tcp on 10.10.10.117
SYN Stealth Scan Timing: About 28.84% done; ETC: 13:16 (0:03:44 remaining)
SYN Stealth Scan Timing: About 37.66% done; ETC: 13:16 (0:03:20 remaining)
SYN Stealth Scan Timing: About 46.88% done; ETC: 13:16 (0:02:51 remaining)
SYN Stealth Scan Timing: About 57.16% done; ETC: 13:16 (0:02:16 remaining)
Discovered open port 6697/tcp on 10.10.10.117
SYN Stealth Scan Timing: About 66.05% done; ETC: 13:16 (0:01:48 remaining)
SYN Stealth Scan Timing: About 75.59% done; ETC: 13:16 (0:01:18 remaining)
SYN Stealth Scan Timing: About 85.67% done; ETC: 13:16 (0:00:45 remaining)
Discovered open port 45010/tcp on 10.10.10.117
Completed SYN Stealth Scan at 13:16, 320.15s elapsed (65535 total ports)
Nmap scan report for 10.10.10.117
Host is up (0.13s latency).
Not shown: 65508 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
111/tcp   open     rpcbind
6686/tcp  filtered unknown
6697/tcp  open     ircs-u
8067/tcp  open     infi-async
9586/tcp  filtered unknown
10232/tcp filtered unknown
14658/tcp filtered unknown
15260/tcp filtered unknown
20891/tcp filtered unknown
25560/tcp filtered unknown
31810/tcp filtered unknown
36746/tcp filtered unknown
37197/tcp filtered unknown
39741/tcp filtered unknown
41814/tcp filtered unknown
45010/tcp open     unknown
45668/tcp filtered unknown
46717/tcp filtered unknown
50999/tcp filtered unknown
53108/tcp filtered unknown
55375/tcp filtered unknown
60868/tcp filtered unknown
61956/tcp filtered unknown
64419/tcp filtered unknown
65534/tcp open     unknown

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 320.27 seconds
           Raw packets sent: 69732 (3.068MB) | Rcvd: 69692 (2.794MB)


Making a script scan on extra ports: 6697, 8067, 45010, 65534
                                                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 13:16 EDT
Nmap scan report for 10.10.10.117
Host is up (0.12s latency).

PORT      STATE SERVICE VERSION
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
45010/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.07 seconds



---------------------Starting Nmap Vulns Scan---------------------
                                                                            
Running CVE scan on all ports
                                                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 13:16 EDT
Nmap scan report for 10.10.10.117
Host is up (0.13s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| vulners: 
|   cpe:/a:openbsd:openssh:6.7p1: 
|       95499236-C9FE-56A6-9D7D-E943A24B633A    10.0    https://vulners.com/githubexploit/95499236-C9FE-56A6-9D7D-E943A24B633A      *EXPLOIT*
|       2C119FFA-ECE0-5E14-A4A4-354A2C38071A    10.0    https://vulners.com/githubexploit/2C119FFA-ECE0-5E14-A4A4-354A2C38071A      *EXPLOIT*
|       CVE-2023-38408  9.8     https://vulners.com/cve/CVE-2023-38408
|       CVE-2016-1908   9.8     https://vulners.com/cve/CVE-2016-1908
|       B8190CDB-3EB9-5631-9828-8064A1575B23    9.8     https://vulners.com/githubexploit/B8190CDB-3EB9-5631-9828-8064A1575B23      *EXPLOIT*
|       8FC9C5AB-3968-5F3C-825E-E8DB5379A623    9.8     https://vulners.com/githubexploit/8FC9C5AB-3968-5F3C-825E-E8DB5379A623      *EXPLOIT*
|       8AD01159-548E-546E-AA87-2DE89F3927EC    9.8     https://vulners.com/githubexploit/8AD01159-548E-546E-AA87-2DE89F3927EC      *EXPLOIT*
|       5E6968B4-DBD6-57FA-BF6E-D9B2219DB27A    9.8     https://vulners.com/githubexploit/5E6968B4-DBD6-57FA-BF6E-D9B2219DB27A      *EXPLOIT*
|       CVE-2015-5600   8.5     https://vulners.com/cve/CVE-2015-5600
|       CVE-2016-0778   8.1     https://vulners.com/cve/CVE-2016-0778
|       PACKETSTORM:140070      7.8     https://vulners.com/packetstorm/PACKETSTORM:140070  *EXPLOIT*
|       EXPLOITPACK:5BCA798C6BA71FAE29334297EC0B6A09    7.8     https://vulners.com/exploitpack/EXPLOITPACK:5BCA798C6BA71FAE29334297EC0B6A09        *EXPLOIT*
|       CVE-2020-15778  7.8     https://vulners.com/cve/CVE-2020-15778
|       CVE-2016-10012  7.8     https://vulners.com/cve/CVE-2016-10012
|       CVE-2015-8325   7.8     https://vulners.com/cve/CVE-2015-8325
|       1337DAY-ID-26494        7.8     https://vulners.com/zdt/1337DAY-ID-26494    *EXPLOIT*
|       SSV:92579       7.5     https://vulners.com/seebug/SSV:92579    *EXPLOIT*
|       PACKETSTORM:173661      7.5     https://vulners.com/packetstorm/PACKETSTORM:173661  *EXPLOIT*
|       F0979183-AE88-53B4-86CF-3AF0523F3807    7.5     https://vulners.com/githubexploit/F0979183-AE88-53B4-86CF-3AF0523F3807      *EXPLOIT*
|       EDB-ID:40888    7.5     https://vulners.com/exploitdb/EDB-ID:40888 *EXPLOIT*
|       CVE-2016-6515   7.5     https://vulners.com/cve/CVE-2016-6515
|       CVE-2016-10708  7.5     https://vulners.com/cve/CVE-2016-10708
|       1337DAY-ID-26576        7.5     https://vulners.com/zdt/1337DAY-ID-26576    *EXPLOIT*
|       CVE-2016-10009  7.3     https://vulners.com/cve/CVE-2016-10009
|       SSV:92582       7.2     https://vulners.com/seebug/SSV:92582    *EXPLOIT*
|       CVE-2021-41617  7.0     https://vulners.com/cve/CVE-2021-41617
|       CVE-2016-10010  7.0     https://vulners.com/cve/CVE-2016-10010
|       PACKETSTORM:151227      0.0     https://vulners.com/packetstorm/PACKETSTORM:151227  *EXPLOIT*
|       PACKETSTORM:140261      0.0     https://vulners.com/packetstorm/PACKETSTORM:140261  *EXPLOIT*
|       PACKETSTORM:138006      0.0     https://vulners.com/packetstorm/PACKETSTORM:138006  *EXPLOIT*
|       PACKETSTORM:137942      0.0     https://vulners.com/packetstorm/PACKETSTORM:137942  *EXPLOIT*
|_      1337DAY-ID-30937        0.0     https://vulners.com/zdt/1337DAY-ID-30937    *EXPLOIT*
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
| vulners: 
|   cpe:/a:apache:http_server:2.4.10: 
|       95499236-C9FE-56A6-9D7D-E943A24B633A    10.0    https://vulners.com/githubexploit/95499236-C9FE-56A6-9D7D-E943A24B633A      *EXPLOIT*
|       2C119FFA-ECE0-5E14-A4A4-354A2C38071A    10.0    https://vulners.com/githubexploit/2C119FFA-ECE0-5E14-A4A4-354A2C38071A      *EXPLOIT*
|       F607361B-6369-5DF5-9B29-E90FA29DC565    9.8     https://vulners.com/githubexploit/F607361B-6369-5DF5-9B29-E90FA29DC565      *EXPLOIT*
|       EDB-ID:51193    9.8     https://vulners.com/exploitdb/EDB-ID:51193 *EXPLOIT*
|       CVE-2024-38476  9.8     https://vulners.com/cve/CVE-2024-38476
|       CVE-2024-38474  9.8     https://vulners.com/cve/CVE-2024-38474
|       CVE-2023-25690  9.8     https://vulners.com/cve/CVE-2023-25690
|       CVE-2022-31813  9.8     https://vulners.com/cve/CVE-2022-31813
|       CVE-2022-23943  9.8     https://vulners.com/cve/CVE-2022-23943
|       CVE-2022-22720  9.8     https://vulners.com/cve/CVE-2022-22720
|       CVE-2021-44790  9.8     https://vulners.com/cve/CVE-2021-44790
|       CVE-2021-39275  9.8     https://vulners.com/cve/CVE-2021-39275
|       CVE-2021-26691  9.8     https://vulners.com/cve/CVE-2021-26691
|       CVE-2018-1312   9.8     https://vulners.com/cve/CVE-2018-1312
|       CVE-2017-7679   9.8     https://vulners.com/cve/CVE-2017-7679
|       CVE-2017-3169   9.8     https://vulners.com/cve/CVE-2017-3169
|       CVE-2017-3167   9.8     https://vulners.com/cve/CVE-2017-3167
|       B02819DB-1481-56C4-BD09-6B4574297109    9.8     https://vulners.com/githubexploit/B02819DB-1481-56C4-BD09-6B4574297109      *EXPLOIT*
|       5C1BB960-90C1-5EBF-9BEF-F58BFFDFEED9    9.8     https://vulners.com/githubexploit/5C1BB960-90C1-5EBF-9BEF-F58BFFDFEED9      *EXPLOIT*
|       3F17CA20-788F-5C45-88B3-E12DB2979B7B    9.8     https://vulners.com/githubexploit/3F17CA20-788F-5C45-88B3-E12DB2979B7B      *EXPLOIT*
|       1337DAY-ID-39214        9.8     https://vulners.com/zdt/1337DAY-ID-39214    *EXPLOIT*
|       CVE-2024-38475  9.1     https://vulners.com/cve/CVE-2024-38475
|       CVE-2022-28615  9.1     https://vulners.com/cve/CVE-2022-28615
|       CVE-2022-22721  9.1     https://vulners.com/cve/CVE-2022-22721
|       CVE-2017-9788   9.1     https://vulners.com/cve/CVE-2017-9788
|       0486EBEE-F207-570A-9AD8-33269E72220A    9.1     https://vulners.com/githubexploit/0486EBEE-F207-570A-9AD8-33269E72220A      *EXPLOIT*
|       CVE-2022-36760  9.0     https://vulners.com/cve/CVE-2022-36760
|       CVE-2021-40438  9.0     https://vulners.com/cve/CVE-2021-40438
|       AE3EF1CC-A0C3-5CB7-A6EF-4DAAAFA59C8C    9.0     https://vulners.com/githubexploit/AE3EF1CC-A0C3-5CB7-A6EF-4DAAAFA59C8C      *EXPLOIT*
|       8AFB43C5-ABD4-52AD-BB19-24D7884FF2A2    9.0     https://vulners.com/githubexploit/8AFB43C5-ABD4-52AD-BB19-24D7884FF2A2      *EXPLOIT*
|       7F48C6CF-47B2-5AF9-B6FD-1735FB2A95B2    9.0     https://vulners.com/githubexploit/7F48C6CF-47B2-5AF9-B6FD-1735FB2A95B2      *EXPLOIT*
|       4810E2D9-AC5F-5B08-BFB3-DDAFA2F63332    9.0     https://vulners.com/githubexploit/4810E2D9-AC5F-5B08-BFB3-DDAFA2F63332      *EXPLOIT*
|       4373C92A-2755-5538-9C91-0469C995AA9B    9.0     https://vulners.com/githubexploit/4373C92A-2755-5538-9C91-0469C995AA9B      *EXPLOIT*
|       36618CA8-9316-59CA-B748-82F15F407C4F    9.0     https://vulners.com/githubexploit/36618CA8-9316-59CA-B748-82F15F407C4F      *EXPLOIT*
|       CVE-2021-44224  8.2     https://vulners.com/cve/CVE-2021-44224
|       B0A9E5E8-7CCC-5984-9922-A89F11D6BF38    8.2     https://vulners.com/githubexploit/B0A9E5E8-7CCC-5984-9922-A89F11D6BF38      *EXPLOIT*
|       CVE-2017-15715  8.1     https://vulners.com/cve/CVE-2017-15715
|       CVE-2016-5387   8.1     https://vulners.com/cve/CVE-2016-5387
|       PACKETSTORM:181038      7.5     https://vulners.com/packetstorm/PACKETSTORM:181038  *EXPLOIT*
|       PACKETSTORM:176334      7.5     https://vulners.com/packetstorm/PACKETSTORM:176334  *EXPLOIT*
|       PACKETSTORM:171631      7.5     https://vulners.com/packetstorm/PACKETSTORM:171631  *EXPLOIT*
|       MSF:AUXILIARY-SCANNER-HTTP-APACHE_OPTIONSBLEED- 7.5     https://vulners.com/metasploit/MSF:AUXILIARY-SCANNER-HTTP-APACHE_OPTIONSBLEED-      *EXPLOIT*
|       F7F6E599-CEF4-5E03-8E10-FE18C4101E38    7.5     https://vulners.com/githubexploit/F7F6E599-CEF4-5E03-8E10-FE18C4101E38      *EXPLOIT*
|       EDB-ID:42745    7.5     https://vulners.com/exploitdb/EDB-ID:42745 *EXPLOIT*
|       EDB-ID:40961    7.5     https://vulners.com/exploitdb/EDB-ID:40961 *EXPLOIT*
|       E5C174E5-D6E8-56E0-8403-D287DE52EB3F    7.5     https://vulners.com/githubexploit/E5C174E5-D6E8-56E0-8403-D287DE52EB3F      *EXPLOIT*
|       DB6E1BBD-08B1-574D-A351-7D6BB9898A4A    7.5     https://vulners.com/githubexploit/DB6E1BBD-08B1-574D-A351-7D6BB9898A4A      *EXPLOIT*
|       CVE-2024-40898  7.5     https://vulners.com/cve/CVE-2024-40898
|       CVE-2024-39573  7.5     https://vulners.com/cve/CVE-2024-39573
|       CVE-2024-38477  7.5     https://vulners.com/cve/CVE-2024-38477
|       CVE-2024-27316  7.5     https://vulners.com/cve/CVE-2024-27316
|       CVE-2023-31122  7.5     https://vulners.com/cve/CVE-2023-31122
|       CVE-2022-30556  7.5     https://vulners.com/cve/CVE-2022-30556
|       CVE-2022-29404  7.5     https://vulners.com/cve/CVE-2022-29404
|       CVE-2022-26377  7.5     https://vulners.com/cve/CVE-2022-26377
|       CVE-2022-22719  7.5     https://vulners.com/cve/CVE-2022-22719
|       CVE-2021-34798  7.5     https://vulners.com/cve/CVE-2021-34798
|       CVE-2021-26690  7.5     https://vulners.com/cve/CVE-2021-26690
|       CVE-2019-0217   7.5     https://vulners.com/cve/CVE-2019-0217
|       CVE-2019-0215   7.5     https://vulners.com/cve/CVE-2019-0215
|       CVE-2018-17199  7.5     https://vulners.com/cve/CVE-2018-17199
|       CVE-2018-1303   7.5     https://vulners.com/cve/CVE-2018-1303
|       CVE-2017-9798   7.5     https://vulners.com/cve/CVE-2017-9798
|       CVE-2017-15710  7.5     https://vulners.com/cve/CVE-2017-15710
|       CVE-2016-8743   7.5     https://vulners.com/cve/CVE-2016-8743
|       CVE-2016-2161   7.5     https://vulners.com/cve/CVE-2016-2161
|       CVE-2016-0736   7.5     https://vulners.com/cve/CVE-2016-0736
|       CVE-2006-20001  7.5     https://vulners.com/cve/CVE-2006-20001
|       C9A1C0C1-B6E3-5955-A4F1-DEA0E505B14B    7.5     https://vulners.com/githubexploit/C9A1C0C1-B6E3-5955-A4F1-DEA0E505B14B      *EXPLOIT*
|       BD3652A9-D066-57BA-9943-4E34970463B9    7.5     https://vulners.com/githubexploit/BD3652A9-D066-57BA-9943-4E34970463B9      *EXPLOIT*
|       B5E74010-A082-5ECE-AB37-623A5B33FE7D    7.5     https://vulners.com/githubexploit/B5E74010-A082-5ECE-AB37-623A5B33FE7D      *EXPLOIT*
|       B0208442-6E17-5772-B12D-B5BE30FA5540    7.5     https://vulners.com/githubexploit/B0208442-6E17-5772-B12D-B5BE30FA5540      *EXPLOIT*
|       A820A056-9F91-5059-B0BC-8D92C7A31A52    7.5     https://vulners.com/githubexploit/A820A056-9F91-5059-B0BC-8D92C7A31A52      *EXPLOIT*
|       A0F268C8-7319-5637-82F7-8DAF72D14629    7.5     https://vulners.com/githubexploit/A0F268C8-7319-5637-82F7-8DAF72D14629      *EXPLOIT*
|       9814661A-35A4-5DB7-BB25-A1040F365C81    7.5     https://vulners.com/githubexploit/9814661A-35A4-5DB7-BB25-A1040F365C81      *EXPLOIT*
|       5A864BCC-B490-5532-83AB-2E4109BB3C31    7.5     https://vulners.com/githubexploit/5A864BCC-B490-5532-83AB-2E4109BB3C31      *EXPLOIT*
|       45D138AD-BEC6-552A-91EA-8816914CA7F4    7.5     https://vulners.com/githubexploit/45D138AD-BEC6-552A-91EA-8816914CA7F4      *EXPLOIT*
|       17C6AD2A-8469-56C8-BBBE-1764D0DF1680    7.5     https://vulners.com/githubexploit/17C6AD2A-8469-56C8-BBBE-1764D0DF1680      *EXPLOIT*
|       1337DAY-ID-38427        7.5     https://vulners.com/zdt/1337DAY-ID-38427    *EXPLOIT*
|       CVE-2020-35452  7.3     https://vulners.com/cve/CVE-2020-35452
|_      PACKETSTORM:140265      0.0     https://vulners.com/packetstorm/PACKETSTORM:140265  *EXPLOIT*
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          45010/tcp   status
|   100024  1          48072/udp6  status
|   100024  1          58879/tcp6  status
|_  100024  1          60060/udp   status
6697/tcp  open  irc     UnrealIRCd (Admin email djmardov@irked.htb)
8067/tcp  open  irc     UnrealIRCd (Admin email djmardov@irked.htb)
45010/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd (Admin email djmardov@irked.htb)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.18 seconds


Running Vuln scan on all ports
                                                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 13:16 EDT
Nmap scan report for 10.10.10.117
Host is up (0.13s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| vulners: 
|   cpe:/a:openbsd:openssh:6.7p1: 
|       95499236-C9FE-56A6-9D7D-E943A24B633A    10.0    https://vulners.com/githubexploit/95499236-C9FE-56A6-9D7D-E943A24B633A      *EXPLOIT*
|       2C119FFA-ECE0-5E14-A4A4-354A2C38071A    10.0    https://vulners.com/githubexploit/2C119FFA-ECE0-5E14-A4A4-354A2C38071A      *EXPLOIT*
|       CVE-2023-38408  9.8     https://vulners.com/cve/CVE-2023-38408
|       CVE-2016-1908   9.8     https://vulners.com/cve/CVE-2016-1908
|       B8190CDB-3EB9-5631-9828-8064A1575B23    9.8     https://vulners.com/githubexploit/B8190CDB-3EB9-5631-9828-8064A1575B23      *EXPLOIT*
|       8FC9C5AB-3968-5F3C-825E-E8DB5379A623    9.8     https://vulners.com/githubexploit/8FC9C5AB-3968-5F3C-825E-E8DB5379A623      *EXPLOIT*
|       8AD01159-548E-546E-AA87-2DE89F3927EC    9.8     https://vulners.com/githubexploit/8AD01159-548E-546E-AA87-2DE89F3927EC      *EXPLOIT*
|       5E6968B4-DBD6-57FA-BF6E-D9B2219DB27A    9.8     https://vulners.com/githubexploit/5E6968B4-DBD6-57FA-BF6E-D9B2219DB27A      *EXPLOIT*
|       CVE-2015-5600   8.5     https://vulners.com/cve/CVE-2015-5600
|       CVE-2016-0778   8.1     https://vulners.com/cve/CVE-2016-0778
|       PACKETSTORM:140070      7.8     https://vulners.com/packetstorm/PACKETSTORM:140070  *EXPLOIT*
|       EXPLOITPACK:5BCA798C6BA71FAE29334297EC0B6A09    7.8     https://vulners.com/exploitpack/EXPLOITPACK:5BCA798C6BA71FAE29334297EC0B6A09        *EXPLOIT*
|       CVE-2020-15778  7.8     https://vulners.com/cve/CVE-2020-15778
|       CVE-2016-10012  7.8     https://vulners.com/cve/CVE-2016-10012
|       CVE-2015-8325   7.8     https://vulners.com/cve/CVE-2015-8325
|       1337DAY-ID-26494        7.8     https://vulners.com/zdt/1337DAY-ID-26494    *EXPLOIT*
|       SSV:92579       7.5     https://vulners.com/seebug/SSV:92579    *EXPLOIT*
|       PACKETSTORM:173661      7.5     https://vulners.com/packetstorm/PACKETSTORM:173661  *EXPLOIT*
|       F0979183-AE88-53B4-86CF-3AF0523F3807    7.5     https://vulners.com/githubexploit/F0979183-AE88-53B4-86CF-3AF0523F3807      *EXPLOIT*
|       EDB-ID:40888    7.5     https://vulners.com/exploitdb/EDB-ID:40888 *EXPLOIT*
|       CVE-2016-6515   7.5     https://vulners.com/cve/CVE-2016-6515
|       CVE-2016-10708  7.5     https://vulners.com/cve/CVE-2016-10708
|       1337DAY-ID-26576        7.5     https://vulners.com/zdt/1337DAY-ID-26576    *EXPLOIT*
|       CVE-2016-10009  7.3     https://vulners.com/cve/CVE-2016-10009
|       SSV:92582       7.2     https://vulners.com/seebug/SSV:92582    *EXPLOIT*
|       CVE-2021-41617  7.0     https://vulners.com/cve/CVE-2021-41617
|       CVE-2016-10010  7.0     https://vulners.com/cve/CVE-2016-10010
|       SSV:92580       6.9     https://vulners.com/seebug/SSV:92580    *EXPLOIT*
|       CVE-2015-6564   6.9     https://vulners.com/cve/CVE-2015-6564
|       1337DAY-ID-26577        6.9     https://vulners.com/zdt/1337DAY-ID-26577    *EXPLOIT*
|       EDB-ID:46516    6.8     https://vulners.com/exploitdb/EDB-ID:46516 *EXPLOIT*
|       EDB-ID:46193    6.8     https://vulners.com/exploitdb/EDB-ID:46193 *EXPLOIT*
|       CVE-2019-6110   6.8     https://vulners.com/cve/CVE-2019-6110
|       CVE-2019-6109   6.8     https://vulners.com/cve/CVE-2019-6109
|       C94132FD-1FA5-5342-B6EE-0DAF45EEFFE3    6.8     https://vulners.com/githubexploit/C94132FD-1FA5-5342-B6EE-0DAF45EEFFE3      *EXPLOIT*
|       10213DBE-F683-58BB-B6D3-353173626207    6.8     https://vulners.com/githubexploit/10213DBE-F683-58BB-B6D3-353173626207      *EXPLOIT*
|       CVE-2023-51385  6.5     https://vulners.com/cve/CVE-2023-51385
|       CVE-2016-0777   6.5     https://vulners.com/cve/CVE-2016-0777
|       EDB-ID:40858    6.4     https://vulners.com/exploitdb/EDB-ID:40858 *EXPLOIT*
|       EDB-ID:40119    6.4     https://vulners.com/exploitdb/EDB-ID:40119 *EXPLOIT*
|       EDB-ID:39569    6.4     https://vulners.com/exploitdb/EDB-ID:39569 *EXPLOIT*
|       CVE-2016-3115   6.4     https://vulners.com/cve/CVE-2016-3115
|       EDB-ID:40136    5.9     https://vulners.com/exploitdb/EDB-ID:40136 *EXPLOIT*
|       EDB-ID:40113    5.9     https://vulners.com/exploitdb/EDB-ID:40113 *EXPLOIT*
|       CVE-2023-48795  5.9     https://vulners.com/cve/CVE-2023-48795
|       CVE-2020-14145  5.9     https://vulners.com/cve/CVE-2020-14145
|       CVE-2019-6111   5.9     https://vulners.com/cve/CVE-2019-6111
|       CVE-2016-6210   5.9     https://vulners.com/cve/CVE-2016-6210
|       EXPLOITPACK:98FE96309F9524B8C84C508837551A19    5.8     https://vulners.com/exploitpack/EXPLOITPACK:98FE96309F9524B8C84C508837551A19        *EXPLOIT*
|       EXPLOITPACK:5330EA02EBDE345BFC9D6DDDD97F9E97    5.8     https://vulners.com/exploitpack/EXPLOITPACK:5330EA02EBDE345BFC9D6DDDD97F9E97        *EXPLOIT*
|       1337DAY-ID-32328        5.8     https://vulners.com/zdt/1337DAY-ID-32328    *EXPLOIT*
|       1337DAY-ID-32009        5.8     https://vulners.com/zdt/1337DAY-ID-32009    *EXPLOIT*
|       SSV:91041       5.5     https://vulners.com/seebug/SSV:91041    *EXPLOIT*
|       PACKETSTORM:140019      5.5     https://vulners.com/packetstorm/PACKETSTORM:140019  *EXPLOIT*
|       PACKETSTORM:136234      5.5     https://vulners.com/packetstorm/PACKETSTORM:136234  *EXPLOIT*
|       EXPLOITPACK:F92411A645D85F05BDBD274FD222226F    5.5     https://vulners.com/exploitpack/EXPLOITPACK:F92411A645D85F05BDBD274FD222226F        *EXPLOIT*
|       EXPLOITPACK:9F2E746846C3C623A27A441281EAD138    5.5     https://vulners.com/exploitpack/EXPLOITPACK:9F2E746846C3C623A27A441281EAD138        *EXPLOIT*
|       EXPLOITPACK:1902C998CBF9154396911926B4C3B330    5.5     https://vulners.com/exploitpack/EXPLOITPACK:1902C998CBF9154396911926B4C3B330        *EXPLOIT*
|       CVE-2016-10011  5.5     https://vulners.com/cve/CVE-2016-10011
|       PACKETSTORM:181223      5.3     https://vulners.com/packetstorm/PACKETSTORM:181223  *EXPLOIT*
|       MSF:AUXILIARY-SCANNER-SSH-SSH_ENUMUSERS-        5.3     https://vulners.com/metasploit/MSF:AUXILIARY-SCANNER-SSH-SSH_ENUMUSERS-     *EXPLOIT*
|       EDB-ID:45939    5.3     https://vulners.com/exploitdb/EDB-ID:45939 *EXPLOIT*
|       EDB-ID:45233    5.3     https://vulners.com/exploitdb/EDB-ID:45233 *EXPLOIT*
|       CVE-2018-20685  5.3     https://vulners.com/cve/CVE-2018-20685
|       CVE-2018-15919  5.3     https://vulners.com/cve/CVE-2018-15919
|       CVE-2018-15473  5.3     https://vulners.com/cve/CVE-2018-15473
|       CVE-2017-15906  5.3     https://vulners.com/cve/CVE-2017-15906
|       CVE-2016-20012  5.3     https://vulners.com/cve/CVE-2016-20012
|       SSH_ENUM        5.0     https://vulners.com/canvas/SSH_ENUM     *EXPLOIT*
|       PACKETSTORM:150621      5.0     https://vulners.com/packetstorm/PACKETSTORM:150621  *EXPLOIT*
|       EXPLOITPACK:F957D7E8A0CC1E23C3C649B764E13FB0    5.0     https://vulners.com/exploitpack/EXPLOITPACK:F957D7E8A0CC1E23C3C649B764E13FB0        *EXPLOIT*
|       EXPLOITPACK:EBDBC5685E3276D648B4D14B75563283    5.0     https://vulners.com/exploitpack/EXPLOITPACK:EBDBC5685E3276D648B4D14B75563283        *EXPLOIT*
|       1337DAY-ID-31730        5.0     https://vulners.com/zdt/1337DAY-ID-31730    *EXPLOIT*
|       SSV:90447       4.6     https://vulners.com/seebug/SSV:90447    *EXPLOIT*
|       EXPLOITPACK:802AF3229492E147A5F09C7F2B27C6DF    4.3     https://vulners.com/exploitpack/EXPLOITPACK:802AF3229492E147A5F09C7F2B27C6DF        *EXPLOIT*
|       EXPLOITPACK:5652DDAA7FE452E19AC0DC1CD97BA3EF    4.3     https://vulners.com/exploitpack/EXPLOITPACK:5652DDAA7FE452E19AC0DC1CD97BA3EF        *EXPLOIT*
|       CVE-2015-5352   4.3     https://vulners.com/cve/CVE-2015-5352
|       1337DAY-ID-25440        4.3     https://vulners.com/zdt/1337DAY-ID-25440    *EXPLOIT*
|       1337DAY-ID-25438        4.3     https://vulners.com/zdt/1337DAY-ID-25438    *EXPLOIT*
|       CVE-2021-36368  3.7     https://vulners.com/cve/CVE-2021-36368
|       SSV:92581       2.1     https://vulners.com/seebug/SSV:92581    *EXPLOIT*
|       CVE-2015-6563   1.9     https://vulners.com/cve/CVE-2015-6563
|       PACKETSTORM:151227      0.0     https://vulners.com/packetstorm/PACKETSTORM:151227  *EXPLOIT*
|       PACKETSTORM:140261      0.0     https://vulners.com/packetstorm/PACKETSTORM:140261  *EXPLOIT*
|       PACKETSTORM:138006      0.0     https://vulners.com/packetstorm/PACKETSTORM:138006  *EXPLOIT*
|       PACKETSTORM:137942      0.0     https://vulners.com/packetstorm/PACKETSTORM:137942  *EXPLOIT*
|_      1337DAY-ID-30937        0.0     https://vulners.com/zdt/1337DAY-ID-30937    *EXPLOIT*
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
| vulners: 
|   cpe:/a:apache:http_server:2.4.10: 
|       95499236-C9FE-56A6-9D7D-E943A24B633A    10.0    https://vulners.com/githubexploit/95499236-C9FE-56A6-9D7D-E943A24B633A      *EXPLOIT*
|       2C119FFA-ECE0-5E14-A4A4-354A2C38071A    10.0    https://vulners.com/githubexploit/2C119FFA-ECE0-5E14-A4A4-354A2C38071A      *EXPLOIT*
|       F607361B-6369-5DF5-9B29-E90FA29DC565    9.8     https://vulners.com/githubexploit/F607361B-6369-5DF5-9B29-E90FA29DC565      *EXPLOIT*
|       EDB-ID:51193    9.8     https://vulners.com/exploitdb/EDB-ID:51193 *EXPLOIT*
|       CVE-2024-38476  9.8     https://vulners.com/cve/CVE-2024-38476
|       CVE-2024-38474  9.8     https://vulners.com/cve/CVE-2024-38474
|       CVE-2023-25690  9.8     https://vulners.com/cve/CVE-2023-25690
|       CVE-2022-31813  9.8     https://vulners.com/cve/CVE-2022-31813
|       CVE-2022-23943  9.8     https://vulners.com/cve/CVE-2022-23943
|       CVE-2022-22720  9.8     https://vulners.com/cve/CVE-2022-22720
|       CVE-2021-44790  9.8     https://vulners.com/cve/CVE-2021-44790
|       CVE-2021-39275  9.8     https://vulners.com/cve/CVE-2021-39275
|       CVE-2021-26691  9.8     https://vulners.com/cve/CVE-2021-26691
|       CVE-2018-1312   9.8     https://vulners.com/cve/CVE-2018-1312
|       CVE-2017-7679   9.8     https://vulners.com/cve/CVE-2017-7679
|       CVE-2017-3169   9.8     https://vulners.com/cve/CVE-2017-3169
|       CVE-2017-3167   9.8     https://vulners.com/cve/CVE-2017-3167
|       B02819DB-1481-56C4-BD09-6B4574297109    9.8     https://vulners.com/githubexploit/B02819DB-1481-56C4-BD09-6B4574297109      *EXPLOIT*
|       5C1BB960-90C1-5EBF-9BEF-F58BFFDFEED9    9.8     https://vulners.com/githubexploit/5C1BB960-90C1-5EBF-9BEF-F58BFFDFEED9      *EXPLOIT*
|       3F17CA20-788F-5C45-88B3-E12DB2979B7B    9.8     https://vulners.com/githubexploit/3F17CA20-788F-5C45-88B3-E12DB2979B7B      *EXPLOIT*
|       1337DAY-ID-39214        9.8     https://vulners.com/zdt/1337DAY-ID-39214    *EXPLOIT*
|       CVE-2024-38475  9.1     https://vulners.com/cve/CVE-2024-38475
|       CVE-2022-28615  9.1     https://vulners.com/cve/CVE-2022-28615
|       CVE-2022-22721  9.1     https://vulners.com/cve/CVE-2022-22721
|       CVE-2017-9788   9.1     https://vulners.com/cve/CVE-2017-9788
|       0486EBEE-F207-570A-9AD8-33269E72220A    9.1     https://vulners.com/githubexploit/0486EBEE-F207-570A-9AD8-33269E72220A      *EXPLOIT*
|       CVE-2022-36760  9.0     https://vulners.com/cve/CVE-2022-36760
|       CVE-2021-40438  9.0     https://vulners.com/cve/CVE-2021-40438
|       AE3EF1CC-A0C3-5CB7-A6EF-4DAAAFA59C8C    9.0     https://vulners.com/githubexploit/AE3EF1CC-A0C3-5CB7-A6EF-4DAAAFA59C8C      *EXPLOIT*
|       8AFB43C5-ABD4-52AD-BB19-24D7884FF2A2    9.0     https://vulners.com/githubexploit/8AFB43C5-ABD4-52AD-BB19-24D7884FF2A2      *EXPLOIT*
|       7F48C6CF-47B2-5AF9-B6FD-1735FB2A95B2    9.0     https://vulners.com/githubexploit/7F48C6CF-47B2-5AF9-B6FD-1735FB2A95B2      *EXPLOIT*
|       4810E2D9-AC5F-5B08-BFB3-DDAFA2F63332    9.0     https://vulners.com/githubexploit/4810E2D9-AC5F-5B08-BFB3-DDAFA2F63332      *EXPLOIT*
|       4373C92A-2755-5538-9C91-0469C995AA9B    9.0     https://vulners.com/githubexploit/4373C92A-2755-5538-9C91-0469C995AA9B      *EXPLOIT*
|       36618CA8-9316-59CA-B748-82F15F407C4F    9.0     https://vulners.com/githubexploit/36618CA8-9316-59CA-B748-82F15F407C4F      *EXPLOIT*
|       CVE-2021-44224  8.2     https://vulners.com/cve/CVE-2021-44224
|       B0A9E5E8-7CCC-5984-9922-A89F11D6BF38    8.2     https://vulners.com/githubexploit/B0A9E5E8-7CCC-5984-9922-A89F11D6BF38      *EXPLOIT*
|       CVE-2017-15715  8.1     https://vulners.com/cve/CVE-2017-15715
|       CVE-2016-5387   8.1     https://vulners.com/cve/CVE-2016-5387
|       PACKETSTORM:181038      7.5     https://vulners.com/packetstorm/PACKETSTORM:181038  *EXPLOIT*
|       PACKETSTORM:176334      7.5     https://vulners.com/packetstorm/PACKETSTORM:176334  *EXPLOIT*
|       PACKETSTORM:171631      7.5     https://vulners.com/packetstorm/PACKETSTORM:171631  *EXPLOIT*
|       MSF:AUXILIARY-SCANNER-HTTP-APACHE_OPTIONSBLEED- 7.5     https://vulners.com/metasploit/MSF:AUXILIARY-SCANNER-HTTP-APACHE_OPTIONSBLEED-      *EXPLOIT*
|       F7F6E599-CEF4-5E03-8E10-FE18C4101E38    7.5     https://vulners.com/githubexploit/F7F6E599-CEF4-5E03-8E10-FE18C4101E38      *EXPLOIT*
|       EDB-ID:42745    7.5     https://vulners.com/exploitdb/EDB-ID:42745 *EXPLOIT*
|       EDB-ID:40961    7.5     https://vulners.com/exploitdb/EDB-ID:40961 *EXPLOIT*
|       E5C174E5-D6E8-56E0-8403-D287DE52EB3F    7.5     https://vulners.com/githubexploit/E5C174E5-D6E8-56E0-8403-D287DE52EB3F      *EXPLOIT*
|       DB6E1BBD-08B1-574D-A351-7D6BB9898A4A    7.5     https://vulners.com/githubexploit/DB6E1BBD-08B1-574D-A351-7D6BB9898A4A      *EXPLOIT*
|       CVE-2024-40898  7.5     https://vulners.com/cve/CVE-2024-40898
|       CVE-2024-39573  7.5     https://vulners.com/cve/CVE-2024-39573
|       CVE-2024-38477  7.5     https://vulners.com/cve/CVE-2024-38477
|       CVE-2024-27316  7.5     https://vulners.com/cve/CVE-2024-27316
|       CVE-2023-31122  7.5     https://vulners.com/cve/CVE-2023-31122
|       CVE-2022-30556  7.5     https://vulners.com/cve/CVE-2022-30556
|       CVE-2022-29404  7.5     https://vulners.com/cve/CVE-2022-29404
|       CVE-2022-26377  7.5     https://vulners.com/cve/CVE-2022-26377
|       CVE-2022-22719  7.5     https://vulners.com/cve/CVE-2022-22719
|       CVE-2021-34798  7.5     https://vulners.com/cve/CVE-2021-34798
|       CVE-2021-26690  7.5     https://vulners.com/cve/CVE-2021-26690
|       CVE-2019-0217   7.5     https://vulners.com/cve/CVE-2019-0217
|       CVE-2019-0215   7.5     https://vulners.com/cve/CVE-2019-0215
|       CVE-2018-17199  7.5     https://vulners.com/cve/CVE-2018-17199
|       CVE-2018-1303   7.5     https://vulners.com/cve/CVE-2018-1303
|       CVE-2017-9798   7.5     https://vulners.com/cve/CVE-2017-9798
|       CVE-2017-15710  7.5     https://vulners.com/cve/CVE-2017-15710
|       CVE-2016-8743   7.5     https://vulners.com/cve/CVE-2016-8743
|       CVE-2016-2161   7.5     https://vulners.com/cve/CVE-2016-2161
|       CVE-2016-0736   7.5     https://vulners.com/cve/CVE-2016-0736
|       CVE-2006-20001  7.5     https://vulners.com/cve/CVE-2006-20001
|       C9A1C0C1-B6E3-5955-A4F1-DEA0E505B14B    7.5     https://vulners.com/githubexploit/C9A1C0C1-B6E3-5955-A4F1-DEA0E505B14B      *EXPLOIT*
|       BD3652A9-D066-57BA-9943-4E34970463B9    7.5     https://vulners.com/githubexploit/BD3652A9-D066-57BA-9943-4E34970463B9      *EXPLOIT*
|       B5E74010-A082-5ECE-AB37-623A5B33FE7D    7.5     https://vulners.com/githubexploit/B5E74010-A082-5ECE-AB37-623A5B33FE7D      *EXPLOIT*
|       B0208442-6E17-5772-B12D-B5BE30FA5540    7.5     https://vulners.com/githubexploit/B0208442-6E17-5772-B12D-B5BE30FA5540      *EXPLOIT*
|       A820A056-9F91-5059-B0BC-8D92C7A31A52    7.5     https://vulners.com/githubexploit/A820A056-9F91-5059-B0BC-8D92C7A31A52      *EXPLOIT*
|       A0F268C8-7319-5637-82F7-8DAF72D14629    7.5     https://vulners.com/githubexploit/A0F268C8-7319-5637-82F7-8DAF72D14629      *EXPLOIT*
|       9814661A-35A4-5DB7-BB25-A1040F365C81    7.5     https://vulners.com/githubexploit/9814661A-35A4-5DB7-BB25-A1040F365C81      *EXPLOIT*
|       5A864BCC-B490-5532-83AB-2E4109BB3C31    7.5     https://vulners.com/githubexploit/5A864BCC-B490-5532-83AB-2E4109BB3C31      *EXPLOIT*
|       45D138AD-BEC6-552A-91EA-8816914CA7F4    7.5     https://vulners.com/githubexploit/45D138AD-BEC6-552A-91EA-8816914CA7F4      *EXPLOIT*
|       17C6AD2A-8469-56C8-BBBE-1764D0DF1680    7.5     https://vulners.com/githubexploit/17C6AD2A-8469-56C8-BBBE-1764D0DF1680      *EXPLOIT*
|       1337DAY-ID-38427        7.5     https://vulners.com/zdt/1337DAY-ID-38427    *EXPLOIT*
|       CVE-2020-35452  7.3     https://vulners.com/cve/CVE-2020-35452
|       FDF3DFA1-ED74-5EE2-BF5C-BA752CA34AE8    6.8     https://vulners.com/githubexploit/FDF3DFA1-ED74-5EE2-BF5C-BA752CA34AE8      *EXPLOIT*
|       0095E929-7573-5E4A-A7FA-F6598A35E8DE    6.8     https://vulners.com/githubexploit/0095E929-7573-5E4A-A7FA-F6598A35E8DE      *EXPLOIT*
|       CVE-2020-1927   6.1     https://vulners.com/cve/CVE-2020-1927
|       CVE-2019-10098  6.1     https://vulners.com/cve/CVE-2019-10098
|       CVE-2019-10092  6.1     https://vulners.com/cve/CVE-2019-10092
|       CVE-2016-4975   6.1     https://vulners.com/cve/CVE-2016-4975
|       CVE-2023-45802  5.9     https://vulners.com/cve/CVE-2023-45802
|       CVE-2018-1302   5.9     https://vulners.com/cve/CVE-2018-1302
|       CVE-2018-1301   5.9     https://vulners.com/cve/CVE-2018-1301
|       1337DAY-ID-33577        5.8     https://vulners.com/zdt/1337DAY-ID-33577    *EXPLOIT*
|       CVE-2020-13938  5.5     https://vulners.com/cve/CVE-2020-13938
|       CVE-2022-37436  5.3     https://vulners.com/cve/CVE-2022-37436
|       CVE-2022-28614  5.3     https://vulners.com/cve/CVE-2022-28614
|       CVE-2022-28330  5.3     https://vulners.com/cve/CVE-2022-28330
|       CVE-2020-1934   5.3     https://vulners.com/cve/CVE-2020-1934
|       CVE-2020-11985  5.3     https://vulners.com/cve/CVE-2020-11985
|       CVE-2019-17567  5.3     https://vulners.com/cve/CVE-2019-17567
|       CVE-2019-0220   5.3     https://vulners.com/cve/CVE-2019-0220
|       CVE-2018-1283   5.3     https://vulners.com/cve/CVE-2018-1283
|       SSV:96537       5.0     https://vulners.com/seebug/SSV:96537    *EXPLOIT*
|       SSV:62058       5.0     https://vulners.com/seebug/SSV:62058    *EXPLOIT*
|       EXPLOITPACK:DAED9B9E8D259B28BF72FC7FDC4755A7    5.0     https://vulners.com/exploitpack/EXPLOITPACK:DAED9B9E8D259B28BF72FC7FDC4755A7        *EXPLOIT*
|       EXPLOITPACK:C8C256BE0BFF5FE1C0405CB0AA9C075D    5.0     https://vulners.com/exploitpack/EXPLOITPACK:C8C256BE0BFF5FE1C0405CB0AA9C075D        *EXPLOIT*
|       CVE-2015-3183   5.0     https://vulners.com/cve/CVE-2015-3183
|       CVE-2015-0228   5.0     https://vulners.com/cve/CVE-2015-0228
|       CVE-2014-3583   5.0     https://vulners.com/cve/CVE-2014-3583
|       CVE-2014-3581   5.0     https://vulners.com/cve/CVE-2014-3581
|       CVE-2013-5704   5.0     https://vulners.com/cve/CVE-2013-5704
|       1337DAY-ID-28573        5.0     https://vulners.com/zdt/1337DAY-ID-28573    *EXPLOIT*
|       1337DAY-ID-26574        5.0     https://vulners.com/zdt/1337DAY-ID-26574    *EXPLOIT*
|       CVE-2016-8612   4.3     https://vulners.com/cve/CVE-2016-8612
|       CVE-2015-3185   4.3     https://vulners.com/cve/CVE-2015-3185
|       CVE-2014-8109   4.3     https://vulners.com/cve/CVE-2014-8109
|       4013EC74-B3C1-5D95-938A-54197A58586D    4.3     https://vulners.com/githubexploit/4013EC74-B3C1-5D95-938A-54197A58586D      *EXPLOIT*
|       1337DAY-ID-33575        4.3     https://vulners.com/zdt/1337DAY-ID-33575    *EXPLOIT*
|_      PACKETSTORM:140265      0.0     https://vulners.com/packetstorm/PACKETSTORM:140265  *EXPLOIT*
|_http-csrf: Couldn't find any CSRF vulnerabilities.
| http-enum: 
|_  /manual/: Potentially interesting folder
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          45010/tcp   status
|   100024  1          48072/udp6  status
|   100024  1          58879/tcp6  status
|_  100024  1          60060/udp   status
6697/tcp  open  irc     UnrealIRCd (Admin email djmardov@irked.htb)
|_ssl-ccs-injection: No reply from server (TIMEOUT)
| irc-botnet-channels: 
|_  ERROR: Closing Link: [10.10.14.3] (Throttled: Reconnecting too fast) -Email djmardov@irked.htb for more information.
8067/tcp  open  irc     UnrealIRCd (Admin email djmardov@irked.htb)
| irc-botnet-channels: 
|_  ERROR: Closing Link: [10.10.14.3] (Throttled: Reconnecting too fast) -Email djmardov@irked.htb for more information.
45010/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd (Admin email djmardov@irked.htb)
| irc-botnet-channels: 
|_  ERROR: Closing Link: [10.10.14.3] (Throttled: Reconnecting too fast) -Email djmardov@irked.htb for more information.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 49.58 seconds



---------------------Recon Recommendations----------------------
                                                                            

Web Servers Recon:
                                                                            
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -l -t 30 -e -k -x .html,.php -u http://10.10.10.117:80 -o recon/gobuster_10.10.10.117_80.txt
nikto -host 10.10.10.117:80 | tee recon/nikto_10.10.10.117_80.txt





Which commands would you like to run?                                       
All (Default), gobuster, nikto, Skip <!>

Running Default in (1) s:  


---------------------Running Recon Commands----------------------
                                                                            

Starting gobuster scan
                                                                            
Error: unknown shorthand flag: 'l' in -l

Finished gobuster scan
                                                                            
=========================
                                                                            
Starting nikto scan
                                                                            
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.10.10.117
+ Target Hostname:    10.10.10.117
+ Target Port:        80
+ Start Time:         2024-09-12 13:18:09 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.10 (Debian)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /: Server may leak inodes via ETags, header found with file /, inode: 48, size: 56c2e413aa86b, mtime: gzip. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ Apache/2.4.10 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ OPTIONS: Allowed HTTP Methods: GET, HEAD, POST, OPTIONS .
+ /manual/: Web server manual found.
+ /manual/images/: Directory indexing found.
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ 8046 requests: 0 error(s) and 8 item(s) reported on remote host
+ End Time:           2024-09-12 13:35:57 (GMT-4) (1068 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

Finished nikto scan
                                                                            
=========================
                                                                            
                                                                            
                                                                            
---------------------Finished all Nmap scans---------------------           
                                                                            

Completed in 28 minute(s) and 11 second(s)
                                                                            
                                                                            
┌──(root㉿kali)-[/opt/nmapAutomator]
└─# nmap -sV -p 6697 10.10.10.117       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 13:43 EDT
Nmap scan report for 10.10.10.117
Host is up (0.13s latency).

PORT     STATE SERVICE VERSION
6697/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.86 seconds
```
### Enumeration
dead ends with apache and rpc. look at twierd prots and unreallRCd 
come up on backdoor repo. https://github.com/Ranger11Danger/UnrealIRCd-3.2.8.1-Backdoor
confiure and run script for user shell
`python3 exploit.py 10.10.10.117 6697 -payload python` 
<img alt="Image" async src="images/Screenshot 2024-09-12 at 1.49.16 PM.png" width="800px"></img>
note that if you lose shell, because of finicky, revert box. It will only work once per session.
stabilize
same guy as before. other user<img alt="Image" async src="images/Screenshot 2024-09-12 at 1.52.07 PM.png" width="800px"></img>
linpeas.sh thru wget
unknowm SUID's that operate out of tmp. viewusers in this case.
update perms and echo in "bash" for a shell

<img alt="Image" async src="images/Screenshot 2024-09-12 at 2.39.55 PM.png" width="800px"></img>
root:42d281c5264de11810dbff5b6dabb2d5
user:28666829bc3170c2d52707d434496c80
`pass:Kab6h+m+bbp2J:HG`
### Initial Access
### Privilege Escalation