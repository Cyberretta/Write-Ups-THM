<p align="center">
  THM : LazyAdmin<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/lazyadmin<br>
  <img src="">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Web enumeration](#web-enumeration)
- [Sweet Rice backup disclosure](#sweet-rice-backup-disclosure)
- [Sweet Rice arbitrary file upload](#sweet-rice-arbitrary-file-upload)
- [Linux enumeration](#linux-enumeration)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

First, let's use [nmap](https://nmap.org/book/man.html) to look for open ports and services on the target :  
![](https://i.imgur.com/U5SySil.jpg)  

## Web enumeration

Let's enumerate port 80. First, we can use [gobuster](https://github.com/OJ/gobuster) to look for files and directories :  
![](https://i.imgur.com/LSC80MK.jpg)  

Let's take a look at the `/content/` directory :  
![](https://i.imgur.com/FQn6lpj.jpg)  

So now we know that the CMS Sweet Rice is installed on the web server. We can use [gobuster](https://github.com/OJ/gobuster) again to scan the `/content/` directory :  
![](https://i.imgur.com/lLS5ctU.jpg)  

We found a directory named `as`, let's take a look at it using a web browser :  
![](https://i.imgur.com/zdaAT0g.jpg)   

We found the login page for Sweet Rice CMS.

## Sweet Rice backup disclosure

By searching for `Sweet Rice` on [Exploit-DB](https://www.exploit-db.com/), we can find a `Backup Disclosure`
on Sweet Rice 1.5.1. Here is the link to the exploit : https://www.exploit-db.com/exploits/40718.  
So we need to go to `http://[TARGET_IP]/content/inc/mysql_backup` to find MySQL backup files :  
![](https://i.imgur.com/CbwJrxv.jpg)  

Let's download this MySQL backup. We may be able to find the login and password hash for Sweet Rice CMS. So let's use `sqlite` to open sqlite cli 
and then `.read [filename]` to read the backup file :  
![](https://i.imgur.com/1JHkHcD.jpg)  

Now we have the username `manager` and the password hash for Sweet Rice. Let's see what hash type is it using [hashes.com](https://hashes.com/en/decrypt/hash) :  
![](https://i.imgur.com/H61NMWI.jpg)  

It found the password ! We can now login to Sweet Rice.

## Sweet Rice arbitrary file upload

On [Exploit-DB](https://www.exploit-db.com/), we can also find an `Arbitrary File Upload` on Sweet Rice 1.5.1. But we can do it manually. It's simple, Sweet Rice 1.5.1 
doesn't properly filter uploaded files. We are able to upload `.php5` files in order to get a reverse shell. Let's do it by going to `Media Center` :  
![](https://i.imgur.com/hm7vUmT.jpg)  

Here we can upload a reverse shell :  
![](https://i.imgur.com/RNpU0TH.jpg)  

Now, we just have to start a listener on our host like so :  
![](https://i.imgur.com/JZqrl2Y.jpg)  

And then we can click on the file we uploaded on the `Media Center` page on Sweet Rice to get a reverse shell :  
![](https://i.imgur.com/2drfB0J.jpg)  

We have a reverse shell ! We can get the `user.txt` file in `/home/itguy`.

## Linux enumeration

Now, if we take a look at the files in `/home/itguy`, we can find a readable file named `backup.pl` :  
![](https://i.imgur.com/zRgL1dm.jpg)  

By looking at our sudo rights, we can notice that we have the right to execute this file as root without password :  
![](https://i.imgur.com/SYLPFfc.jpg)  

Let's see what's in this file :  
![](https://i.imgur.com/XYgSoBP.jpg)  

It executes a bash script at `/etc/copy.sh`. Let's see what's in this script :  
![](https://i.imgur.com/JgK1gBU.jpg)  

There is a reverse shell. Do we have write permissions on this file ?  
![](https://i.imgur.com/z032Kwn.jpg)  

Yes ! We can write on this file !

## Privilege escalation

Yes ! We just have to edit the IP adress and the port in order to get a reverse shell as root ! Then, we can start a netcat listener, and finally, we have to executre 
the backup script using `sudo /usr/bin/perl /home/itguy/backup.pl` :  
![](https://i.imgur.com/MlG2WAP.jpg)  

## Conclusion

In this room, we practiced : 
- Services an ports enumeration using [nmap](https://nmap.org/book/man.html)
- Website enumeration using [gobuster](https://github.com/OJ/gobuster)
- Sweet Rice 1.5.1 exploitation
- Linux enumeration
- Linux privilege escalation using backup scripts

Thanks for this room and thanks for reading my write ups !












