<p align="center">
  THM : Corridor<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/corridor<br>
  <img src="https://i.imgur.com/kyBIjX6.png">
</p>

## Summary
- [Introduction](#introduction)
- [Website enumeration](#website-enumeration)
- [Hash cracking](#hash-cracking)
- [Making a custom wordlist](#making-a-custom-wordlist)
- [Get the flag](#get-the-flag)
- [Conclusion](#conclusion)

## Introduction

```
You have found yourself in a strange corridor. Can you find your way back to where you came?

In this challenge, you will explore potential IDOR vulnerabilities. Examine the URL endpoints you access as you navigate the website 
and note the hexadecimal values you find (they look an awful lot like a hash, don't they?). This could help you uncover website 
locations you were not expected to access.
```

What is an IDOR ? 

From [Portswigger.net](https://portswigger.net/web-security/access-control/idor)
Insecure direct object references (IDOR) are a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly. The term IDOR was popularized by its appearance in the OWASP 2007 Top Ten. However, it is just one example of many access control implementation mistakes that can lead to access controls being circumvented. IDOR vulnerabilities are most commonly associated with horizontal privilege escalation, but they can also arise in relation to vertical privilege escalation. 

## Website enumeration

Let's take a look at the website :  
![](https://i.imgur.com/3evVlil.png)  

There is some doors where we can click, let's see what happens when we click one of these doors :  
![](https://i.imgur.com/qeH3Hcs.png)  

We are redirected to a page using a hash in the URL. It looks like an MD5 hash. Let's check this out using [Hashes.com](https://hashes.com/en/tools/hash_identifier) :  
![](https://i.imgur.com/0Gi4gvc.png)  

We know this is a MD5 hash. Let's try to crack it.

## Hash cracking

To crack this MD5 hash, I used the `directory-list-2.3-medium.txt` wordlist located in `/usr/share/wordlist/dirbuster` by default in Kali Linux.
```
──(attacker㉿AttackBox)-[~]
└─$ hashcat -a 0 -m 0 8f14e45fceea167a5a36dedd4bea2543 /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -w 3
hashcat (v6.2.5) starting

OpenCL API (OpenCL 2.1 AMD-APP (3110.6)) - Platform #1 [Advanced Micro Devices, Inc.]
=====================================================================================
* Device #1: Radeon RX 580 Series, 7616/7719 MB (4048 MB allocatable), 36MCU

OpenCL API (OpenCL 3.0 PoCL 3.0+debian  Linux, None+Asserts, RELOC, LLVM 13.0.1, SLEEF, DISTRO, POCL_DEBUG) - Platform #2 [The pocl project]
============================================================================================================================================
* Device #2: pthread-AMD Ryzen 3 3100 4-Core Processor, skipped
...
...
8f14e45fceea167a5a36dedd4bea2543:7                        
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 8f14e45fceea167a5a36dedd4bea2543
Time.Started.....: Sat Oct  1 11:29:45 2022 (0 secs)
Time.Estimated...: Sat Oct  1 11:29:45 2022 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 59817.1 kH/s (0.83ms) @ Accel:1024 Loops:1 Thr:64 Vec:1
Recovered........: 1/1 (100.00%) Digests
Progress.........: 220560/220560 (100.00%)
Rejected.........: 0/220560 (0.00%)
Restore.Point....: 0/220560 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: # directory-list-2.3-medium.txt -> nt4stopc
Hardware.Mon.#1..: Temp: 49c Fan:  0% Util:  4% Core:1300MHz Mem:2000MHz Bus:16

Started: Sat Oct  1 11:29:42 2022
Stopped: Sat Oct  1 11:29:46 2022
```
We successfuly cracked the hash. Now we know the page exists in this 
wordlist. The idea I have is to create a new wordlist containing all the MD5 hashes of the original wordlist and then use it with gobuster to find other pages.  

## Making a custom wordlist

To make a custom wordlist, I simply made a little python script :  
```
#!/usr/bin/python3
import hashlib

file = open('/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt', 'r')
text = file.read()

lines = text.split('\n')

for index, line in enumerate(lines):
    result = hashlib.md5(line.encode('utf-8')).hexdigest()
    print(result)

file.close()
```
This script will take every line of the wordlist and print its MD5 hash. Then I make the script executable with `chmod +x script.py` and I run the script using `./script.py > custom_wordlist.txt`. Now 
we have our custom wordlist containing the MD5 hashes.

## Get the flag

Now I use [GoBuster](https://github.com/OJ/gobuster) to find other pages on the website :  
```
┌──(attacker㉿AttackBox)-[~]
└─$ gobuster dir -u http://10.10.41.208/ -w custom_wordlist.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.41.208/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                custom_wordlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/10/01 11:39:27 Starting gobuster in directory enumeration mode
===============================================================
/c20ad4d76fe97759aa27a0c99bff6710 (Status: 200) [Size: 632]
/6512bd43d9caa6e02c990b0a82652dca (Status: 200) [Size: 632]
/d3d9446802a44259755d38e6d163e820 (Status: 200) [Size: 632]
/c4ca4238a0b923820dcc509a6f75849b (Status: 200) [Size: 632]
/c81e728d9d4c2f636f067f89cc14862c (Status: 200) [Size: 632]
/eccbc87e4b5ce2fe28308fd9f2a7baf3 (Status: 200) [Size: 632]
/c51ce410c124a10e0db5e4b97fc2af39 (Status: 200) [Size: 632]
/a87ff679a2f3e71d9181a67b7542122c (Status: 200) [Size: 632]
/e4da3b7fbbce2345d7772b0674a318d5 (Status: 200) [Size: 632]
/1679091c5a880faf6fb5e6087eb1b2dc (Status: 200) [Size: 632]
/45c48cce2e2d7fbdea1afc51c7c6ad26 (Status: 200) [Size: 632]
/8f14e45fceea167a5a36dedd4bea2543 (Status: 200) [Size: 632]
/cfcd208495d565ef66e7dff9f98764da (Status: 200) [Size: 797] <----- Size is different from other pages so it took my attention
/c9f0f895fb98ab9159f51fd0297e236d (Status: 200) [Size: 632]
```

You can see multiple results with the same size, but there is one page that has a different size so it took my attention. If we go to this page :  
![](https://i.imgur.com/YAD9knX.png)  

And we have the flag !
(The hash of this page is simply the MD5 hash of `0`)

## Conclusion

It was a fun room to start the day ! Thanks for this room ! If you want to learn more about IDOR, there is a course on TryHackMe on this subject, [here](https://tryhackme.com/room/idor).
