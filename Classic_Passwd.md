<p align="center">
  THM : Classic Passwd<br>
  Difficulty : Medium<br>
  <img src="https://i.imgur.com/GqkQLNS.png">
</p>

## Summary

- [File command](#file-command)
- [Analyse the file using ghidra](#analyse-the-file-using-ghidra)
- [Conclusion](#conclusion)

## File command

First, I used ``file Challenge.Challenge`` to know what type of file it is.  
```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/Classic_Passwd]
└─$ file Challenge.Challenge 
Challenge.Challenge: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=b80ce38cb25d043128bc2c4e1e122c3d4fbba7f7, for GNU/Linux 3.2.0, not stripped
```

## Analyse the file using ghidra

Looking at the functions in the executable, we can see 3 interesting functions :  
- main  
- gfl  
- vuln  
 
![alt text](https://i.imgur.com/nAt4YoD.png)

First, let's see the main function :  
![alt text](https://i.imgur.com/w0AuTpc.png)

We see in the main function (this function is the entry point of the program), that there is 2 functions called.
vuln() and then gfl(). I think gfl is for 'get flag', and obviously, vuln stands for vulnerabilitie.
Let's take a look at the vuln function :
![alt text](https://i.imgur.com/sVpdbX8.png)

So what is happening in this function ?  
First, there is some variables declaration... we can skip this for the moment.  
Then, the program asks a username to the user using the scanf function.  
The program copies the variable local\_238 (which contains the string the user entered before) to the variable local\_2c8.  
Then, it compares the variable local_2c8 to the variable local_246 (so we can assume that local_246 contains the right username), if they are equal, the program prints "Welcome", else, it prints "Authentification Error"

### 1st method  

Knowing that local_246 contains the right username, we can look at his value and try to decode it.
So local_246 is equal to 0x6435736a36424741. But remember, this is an ELF 64-bit **LSB** pie executable, which means that we have to start interpreting this value from the least significant bit.  
So let's reverse it like so : 41 47 42 36 6a 73 35 64
Now let's decode this to ASCII : AGB6js5d
If you try to use this as a username, it won't work. Why ? Because it's only a part of the username to find.  
Looking at the C code in ghidra don't help... So let's take a look at the assembly code.  
![](https://i.imgur.com/a08fWIv.png)  
We can see that there is 3 hex values here, the first one is the one we already decoded, and the two others are maybe two other parts of the username ?
Let's try to decode those two values and see if it works :  
0x476b6439 -> 9dkG  
0x37 -> 7  
So if we take the 3 values we found and put them together, we have **AGB6js5d9dkG7**.  
Let's use it in the program to get the flag !  
```
attacker@AttackBox:~/Bureau/CTF/Classic_Passwd$ ./Challenge.Challenge 
Insert your username: AGB6js5d9dkG7

Welcome
THM{*********}
attacker@AttackBox:~/Bureau/CTF/Classic_Passwd$
```

**Honestly, I tried to understand why and how it was working like this (the username is divided in 3 parts but looking at the strcmp function call, there is no way to know that there is two other variables used with local_246).  
So for now I will just write what I found and what I understood. I will try again to understand more what is happening in this program, but another time**

### 2nd method

We can use ltrace to recover the value of local_246 variable (the variable that contains the right username).  
I used ``ltrace ./Challenge.Challenge``. When the program asked for a username, I just entered a random string "test". And I got the value of variable local_246 !  
```
attacker@AttackBox:~/Bureau/CTF/Classic_Passwd$ ltrace ./Challenge.Challenge 
printf("Insert your username: ")                                         = 22
__isoc99_scanf(0x55b7e298d01b, 0x7ffda64a7a60, 0, 0Insert your username: test
)                     = 1
strcpy(0x7ffda64a79d0, "test")                                           = 0x7ffda64a79d0
strcmp("test", "AGB6js5d9dkG7")                                          = 51
puts("\nAuthentication Error"
Authentication Error
)                                           = 22
exit(0 <no return ...>
+++ exited (status 0) +++
```

Now we have the right username which is **AGB6js5d9dkG7** !  
To get the flag, we just have to execute the binary like this : ``./Challenge.Challenge``.  
When the executable asks for a username, we just need to enter **AGB6js5d9dkG7**, and we have the flag !
```
attacker@AttackBox:~/Bureau/CTF/Classic_Passwd$ ./Challenge.Challenge 
Insert your username: AGB6js5d9dkG7

Welcome
THM{*********}
attacker@AttackBox:~/Bureau/CTF/Classic_Passwd$
```

### 3rd method

There is a 3rd method to get the flag, but this time without finding the right username.  
Let's take a look at the gfl function :  
![alt text](https://i.imgur.com/URXN6kO.png)  

Let's resume what happens in this function :
There is some variables declarations of course.. after that, there is a while loop.  
In this while loop, if the variable local_c is greater than 0x77d088, we exit the gfl function and it returns nothing because it's a void function.  
Then, if the local_c variable is equal to 0x638a78, we enter in a for loop.  
At the beginning of the for loop, we give the local_10 variable an initial value of 0x1474, and since it is lesser than 9999, we increase this variable by 1 and we continue to go trought the for loop.  
In the for loop, we check if local_10 is equal to 0x2130, if true, we print something in the console.  

Let's take a closer look at this print. It looks like it is the flag we are looking for.  
The two hexadecimal values passed as arguments in the function are interpreted as integer values (because of the '%d' in the print), so let's try to convert them to integer values.

The first one (0x638a78) is equal to 6523512  
The second (0x2130) is equal to 8496

So we have the flag : **THM{*HIDDEN*}** !

## Conclusion
This room was very cool. It was not necessary to use ghidra to get the flag but, it is always useful and interesting to analyse the program to learn more. I'm not very good at reverse engineering for now so it was interesting to train myself on this room.
