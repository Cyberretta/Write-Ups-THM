<p align="center">
  THM : Reversing ELF<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/reverselfiles<br>
  <img src="https://i.imgur.com/I9QGaYJ.jpg">
</p>

## Summary
- [Introduction](#introduction)
- [Crackme 1](#crackme-1)
- [Crackme 2](#crackme-2)
- [Crackme 3](#crackme-3)
- [Crackme 4](#crackme-4)
- [Crackme 5](#crackme-5)
- [Crackme 6](#crackme-6)
- [Crackme 7](#crackme-7)
- [Crackme 8](#crackme-8)
- [Conclusion](#conclusion)

## Introduction

In this room, we will be reversing ELF files using [ltrace](https://man7.org/linux/man-pages/man1/ltrace.1.html) for the easiest tasks, and [Ghidra](https://ghidra-sre.org/) for the more complex tasks. I will explain how I got the flag for every "Crackme" in this room. 
I created a project named Reversing_ELF and I put every task files of this room in this project :  
![](https://i.imgur.com/EEWa5WP.jpg)

## Crackme 1

For the first Crackme, we can simply run the binary (be careful, you should never run an unknown binary without looking at the source code or without 
analysing it using tools like [Ghidra](https://ghidra-sre.org/), of course it's not a milicious file, but it's a good habit to have) :  
```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ chmod +x index.crackme1
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ./index.crackme1 
flag{****************************}
```

## Crackme 2

Same as the first crackme, but this time we need a password :  
```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ./index.crackme2 
Usage: ./index.crackme2 password
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ./index.crackme2 test
Access denied.
```

We can try to use [ltrace](https://man7.org/linux/man-pages/man1/ltrace.1.html), if the binary just makes a string comparison between the password we enter and the good password already 
present in the program, ltrace will show us the two strings compared :  
```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ltrace ./index.crackme2 test
__libc_start_main(0x804849b, 2, 0xff995094, 0x80485c0 <unfinished ...>
strcmp("test", "super_secret_password")                                                                              = 1
puts("Access denied."Access denied.
)                                                                                               = 15
+++ exited (status 1) +++
```

We see that the password we entered "test" is compared with the good password "super_secret_password" ! We can now use this password to get the flag :  
```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ./index.crackme2 super_secret_password
Access granted.
flag{**********************************************}
```

## Crackme 3

For this Crackme, we need to use [Ghidra](https://ghidra-sre.org/)(or any other tools like [IDA Free](https://hex-rays.com/ida-free/), [Cutter](https://cutter.re/)...). The first thing I always look for is the entry point of the program. 
If you take a look at the functions in the program, you will see one named "entry", that's the entry point of the program (where the program starts basically) :  
![](https://i.imgur.com/lgyEJ1X.jpg)  
![](https://i.imgur.com/0LrIYPt.jpg)  

You see in the C code of the "entry" function, that there is another function called named "FUN_080484f4". We can double click this function and take a look at its C code :  
![](https://i.imgur.com/Iw4uE5E.jpg)  

You can see a function call "strcmp". It's a function that compare two strings, and return true if they are equal, or false if they are not. It seems that the two compared strings are 
the password we enter in the program, and a base64 encoded string. Let's copy this base64 string and decode it :  
```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ echo 'ZjByX3kwdXJfNWVjMG5kX2xlNTVvbl91bmJhc2U2NF80bGxfN2gzXzdoMW5nNQ==' | base64 -d && echo ""
****************************************************
```

## Crackme 4

For the fourth Crackme, we can also use [ltrace](https://man7.org/linux/man-pages/man1/ltrace.1.html) :  
```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ltrace ./index.crackme4 test

__libc_start_main(0x400716, 2, 0x7ffd26aafdf8, 0x400760 <unfinished ...>

strcmp("******************", "test")                                                                                 = -7

printf("password "%s" not OK\n", "test"password "test" not OK

)                                                                             = 23

+++ exited (status 0) +++
```

## Crackme 5

**Question  : What will be the input of the file to get output Good game ?**  

```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ltrace ./index.crackme5
__libc_start_main(0x400773, 1, 0x7fff45a237b8, 0x4008d0 <unfinished ...>
puts("Enter your input:"Enter your input:
)                                                                                            = 18
__isoc99_scanf(0x400966, 0x7fff45a23670, 0, 0x7f812d3c58f3test
)                                                          = 1
strlen("test")                                                                                                       = 4
strlen("test")                                                                                                       = 4
strlen("test")                                                                                                       = 4
strlen("test")                                                                                                       = 4
strlen("test")                                                                                                       = 4
strncmp("test", "OfdlDS***************", 28)                                                                  = 37
puts("Always dig deeper"Always dig deeper
)                                                                                            = 18
+++ exited (status 0) +++
```

We can see the string we entered is compared with `OfdlDS****************`. Let's restart the same command but this time we will enter OfdlDS*****************

```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ltrace ./index.crackme5
__libc_start_main(0x400773, 1, 0x7ffe73ba58a8, 0x4008d0 <unfinished ...>
puts("Enter your input:"Enter your input:
)                                                                                            = 18
__isoc99_scanf(0x400966, 0x7ffe73ba5760, 0, 0x7ff7f19f78f3OfdlDS*****************
)                                                          = 1
...
...
strncmp("OfdlDS*****************", "OfdlDS*****************", 28)                                          = 0
puts("Good game"Good game
)                                                                                                    = 10
+++ exited (status 0) +++
```

And we see that the program prints "Good game", so we have the answer : OfdlDS*****************

## Crackme 6

This time, [ltrace](https://man7.org/linux/man-pages/man1/ltrace.1.html) will not help us. We will need to use a tool like [Ghidra](https://ghidra-sre.org/). This time, the entry point is a function named "main" :  
![](https://i.imgur.com/e5kGmQf.jpg)  
![](https://i.imgur.com/qGgLepY.jpg)  

We see a function named "compare_pwd" that is called. Let's take a look at it by double clicking it :  
![](https://i.imgur.com/8IALoV7.jpg)  

Now we see another function named "my_secure_test" that is called, same as before, let's double click it to check the source code :  
![](https://i.imgur.com/2Nagq4b.jpg)  

Here we are ! We can see that every characters of the entered parameter is compared with a plaintext character. The first character of the parameter we enter 
is compared with 1, the second with 3, the third with 3... If we put them together we have the password :  
```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ./index.crackme6 133*****
password OK
```

## Crackme 7

Like for the previous Crackme (6), we have to take a look in the main function :  
![](https://i.imgur.com/66lWRiN.jpg)

As you can see, the program is going to ask for three choices, the choice we make is stored in "local_14". If "local_14" equals 0x7a69, 
the program prints "Wow such h4x0r!" and then call a function named giveFlag(). We can convert the hexadecimal value to an integer with [this tool](https://www.rapidtables.com/convert/number/hex-to-decimal.html) :  
![](https://i.imgur.com/orM1qaX.jpg)  

Now, let's start the program, and enter 31337 so the value 31337 is stored in the "local_14" variable :  
```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ./index.crackme7
Menu:

[1] Say hello
[2] Add numbers
[3] Quit

[>] 31337
Wow such h4x0r!
flag{************************}

```

## Crackme 8

Again, for this Crackme, we need to take a look at the "main" function :  
![](https://i.imgur.com/kjNP8GY.jpg)  

The value we enter as a parameter when executing the program is converted from a string to an integer with the function "atoi". Then, the program checks if the value entered 
equals -0x35010ff3 (It's a negative hexadecimal value). If it does, the program prints "Access granted." and make a call to the "giveFlag" function. Let's convert the 
hexadecimal value to an integer with [this tool](https://www.rapidtables.com/convert/number/hex-to-decimal.html) :  
![](https://i.imgur.com/o8Iu1df.jpg)  

Now, we can execute the crackme8 and use -889262067 as a parameter :  
```
attacker@AttackBox:~/Documents/THM/CTF/Reversing_ELF$ ./index.crackme8 -889262067
Access granted.
flag{*********************************}
```
## Conclusion

I think this room is great for beginners who have never done reverse engineering, and also, this room is useful to learn some basics of C programming language. Since I already know 
the basics of C programming, this room was not that hard for me, but it was useful to train myself on basic reverse engineering. Thanks for this room ! And thanks for reading my write ups !
