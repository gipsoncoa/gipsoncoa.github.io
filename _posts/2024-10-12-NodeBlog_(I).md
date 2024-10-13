---
title: NodeBlog - Hack the Box
date: 2024-10-13 12:37:45 -0400
categories: [Hack the Box]
tags: [Walkthrough]
image: /assets/NodeBlog.png
---
### Scan
```
└─# nmap -sC -sV --open -p- 10.10.11.139
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-11 20:18 EDT
Nmap scan report for 10.10.11.139
Host is up (0.13s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
|_  256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
5000/tcp open  http    Nodjs (Express middleware)
|_http-title: Blog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### Enumeration

### Initial Access
### Privilege Escalation
