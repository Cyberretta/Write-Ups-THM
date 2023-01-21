<p align="center">
  THM : 0x41haz<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/0x41haz<br>
  <img src="https://i.imgur.com/ODoteNT.jpg">
</p>

## Summary

- [Introduction](#introduction)
- [Reversing the binary](#reversing-the-binary)
- [Conclusion](#conclusion)

## Introduction

```
In this challenge, you are asked to solve a simple reversing solution. Download and analyze the binary to discover the password.

There may be anti-reversing measures in place!
```

So we have to analyze a binary to find a password.

## Reversing the binary

First, let's use the `file` command to get basic informations about the binary :  
```
attacker@AttackBox:~/Documents/THM/CTF/0x41haz$ file 0x41haz.0x41haz 
0x41haz.0x41haz: ELF 64-bit MSB *unknown arch 0x3e00* (SYSV)
```

We know it's an ELF file 64-bit MSB but we don't know the arch. I search `"unknown arch 0x3e00" elf` on Google, and I found [this page](https://pentester.blog/?p=247) :  
![](https://i.imgur.com/IukDpeM.jpg)  

The author gives a solution to this error :  
![](https://i.imgur.com/q4x5QUN.jpg)  

Let's try this and see if it works :  
```
attacker@AttackBox:~/Documents/THM/CTF/0x41haz$ objdump -M intel -D ./0x41haz.0x41haz 
objdump: ./0x41haz.0x41haz: format de fichier non reconnu
```

It says that the file format is not recognized. There is another trick explained by the author of the blog :  
![](https://i.imgur.com/EXFsOAz.jpg)  

Let's try to change the 6th byte of the ELF file using [hexedit](https://linux.die.net/man/1/hexedit) :  
![](https://i.imgur.com/0NQdbPH.jpg)  

So I changed the 6th byte from `0x02` to `0x01`. Let's try again :  
```
attacker@AttackBox:~/Documents/THM/CTF/0x41haz$ objdump -M intel -D ./0x41haz.0x41haz 

./0x41haz.0x41haz:     format de fichier elf64-x86-64


Déassemblage de la section .interp :

00000000000002a8 <.interp>:
 2a8:	2f                   	(bad)  
 2a9:	6c                   	ins    BYTE PTR es:[rdi],dx
 2aa:	69 62 36 34 2f 6c 64 	imul   esp,DWORD PTR [rdx+0x36],0x646c2f34
 2b1:	2d 6c 69 6e 75       	sub    eax,0x756e696c
 2b6:	78 2d                	js     2e5 <puts@plt-0xd4b>
 2b8:	78 38                	js     2f2 <puts@plt-0xd3e>
 2ba:	36 2d 36 34 2e 73    	ss sub eax,0x732e3436
 2c0:	6f                   	outs   dx,DWORD PTR ds:[rsi]
 2c1:	2e 32 00             	xor    al,BYTE PTR cs:[rax]

Déassemblage de la section .note.gnu.build-id :

00000000000002c4 <.note.gnu.build-id>:
 2c4:	04 00                	add    al,0x0
 2c6:	00 00                	add    BYTE PTR [rax],al
 2c8:	14 00                	adc    al,0x0
 2ca:	00 00                	add    BYTE PTR [rax],al
 2cc:	03 00                	add    eax,DWORD PTR [rax]
 2ce:	00 00                	add    BYTE PTR [rax],al
 2d0:	47                   	rex.RXB
 2d1:	4e 55                	rex.WRX push rbp
 2d3:	00 6c 9f 2e          	add    BYTE PTR [rdi+rbx*4+0x2e],ch
 2d7:	85 b6 4d 4f 12 b9    	test   DWORD PTR [rsi-0x46edb0b3],esi
 ...
 ...
 ...
 ...
```

It works ! Let's try to use ghidra now. If we look at the function `FUN_00101165`, we notice that this is the function that checks if the password is good or wrong :  
![](https://i.imgur.com/7MKteq1.jpg)  

The line `if (*(char *)((long)&local_1e + (long)local_c) != local_48[local_c]) break;` seems to compare the string we enter and the good password characters one by one. 
It seems that the variable containing the good password is local_1e. Let's decode its value to ascii (This program is an MSB executable, so we have to reverse the 
hexadecimal code) :  
![](https://i.imgur.com/RKrgOoN.jpg)  

It's only a part of the password. If you look at the condition of the while loop, you will notice that it will iterate 13 times since `local_c` starts from 0 and increment 
by 1 and the while loop stops when `local_c` is bigger than 12 (0xc).  

Since the while loop checks for every characters of the password one by one, it means that the password 
is 13 bytes long. But we only have 8 bytes for now. Since the comparison is made by using a pointer, it means that if the program tries to get a byte from `local_1e` by 
using an index bigger than the size of `local_1e`, it will read bytes from memory after `local_1e`.  

To make it simple, it will read every bytes from `local_1e`, then every bytes from 
`local_16` (since this variable is just after `local_1e` in memory), and then it will read every bytes from `local_12`. It will make exactly 13 bytes. So the password must be 
a concatenation of `local_1e` + `local_16` + `local_12`. Let's convert this hexadecimal value (like before, we have to reverse each hexadecimal values separately) :  
![](https://i.imgur.com/m8vBXwP.jpg)  

Now let's copy this string and try to use it as password :  
```
ttacker@AttackBox:~/Documents/THM/CTF/0x41haz$ ./0x41haz.0x41haz 
=======================
Hey , Can You Crackme ?
=======================
It's jus a simple binary 

Tell Me the Password :
2@@25$gfsT&@L
Well Done !!
```

It works ! So we have the password now. The flag is in the format `THM{PASSWORD}`. Let's enter it to finish this room !

## Conclusion

This room was not that easy. I learned some methods to obfuscate ELF binaries, and how to bypass them. This room also trained me to reverse a binary. Thanks for this room 
and thanks for reading my write ups !


