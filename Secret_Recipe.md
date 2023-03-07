<p align="center">
  THM : Secret Recipe<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/registry4n6<br>
  <img src="https://i.imgur.com/rY0vbDD.jpg">
</p>

## Summary

- [Introduction](#introduction)
- [Question 1](#question-1)
- [Question 2](#question-2)
- [Question 3](#question-3)
- [Question 4](#question-4)
- [Question 5](#question-5)
- [Question 6](#question-6)
- [Question 7](#question-7)
- [Question 8](#question-8)
- [Question 9](#question-9)
- [Question 10](#question-10)
- [Question 11](#question-11)
- [Question 12](#question-12)
- [Question 13](#question-13)
- [Question 14](#question-14)
- [Question 15](#question-15)
- [Question 16](#question-16)
- [Question 17](#question-17)
- [Question 18](#question-18)

## Introduction

Jasmine owns a famous New York coffee shop Coffely which is famous city-wide for its unique taste. Only Jasmine keeps the original copy of the recipe, and she only keeps it on her work laptop. Last week, James from the IT department was consulted to fix Jasmine's laptop. But it is suspected he may have copied the secret recipes from Jasmine's machine and is keeping them on his machine.Image showing a Laptop with a magnifying glass
His machine has been confiscated and examined, but no traces could be found. The security department has pulled some important registry artifacts from his device and has tasked you to examine these artifacts and determine the presence of secret files on his machine.

## Question 1

How many files are available in the `Artifacts` folder on the Desktop ?  
![](https://i.imgur.com/uRRzmPC.jpg)  
**Answer : 6**

## Question 2

What is the Computer Name of the Machine found in the registry ?  
![](https://i.imgur.com/gDirQ5y.jpg)  
Answer in `SYSTEM\ControlSet001\Control\ComputerName\ComputerName`  
**Answer : JAMES**

## Question 3

When was the Administrator account created on this machine ? (Format: yyyy-mm-dd hh:mm:ss)  
![](https://i.imgur.com/jhGW7gm.jpg)  
Answer in `SAM\SAM\Domains\Account\Users`  
**Answer : 2021-03-17 14:58:48**

## Question 4

What is the RID associated with the Administrator account ?  
![](https://i.imgur.com/RdIQmtp.jpg)  
Answer in `SAM\SAM\Domains\Account\Users`  
**Answer : 500**

## Question 5

How many User accounts were observed on this machine ?  
Answer in `SAM\SAM\Domains\Account\Users\Names`  
![](https://i.imgur.com/6Nkw8Na.jpg)  
**Answer : 7**

## Question 6

There seems to be a suspicious account created as a backdoor with RID 1013. What is the Account Name ?  
Answer in `SAM\SAM\Domains\Account\Users`  
![](https://i.imgur.com/GUBjFLe.jpg)  
**Answer : bdoor**

## Question 7

What is the VPN connection this host connected to ?  
Answer in `SYSTEM\ControlSet001\Services\bam\State\UserSettings\S-1-5-21-1966530601-318510712-10604624-500`  
![](https://i.imgur.com/FPpqDq3.jpg)  
**Answer : ProtonVPN**

## Question 8

When was the first VPN connection observed ? (Format: YYYY-MM-DD HH:MM:SS)  
Answer in `SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList`  
![](https://i.imgur.com/O7UyTYE.jpg)  
**Answer : 2022-10-12 19:52:36**

## Question 9

There were three shared folders observed on his machine. What is the path of the third share ?  
Answer in `SYSTEM\ControlSet001\Services\LanmanServer\Shares`  
![](https://i.imgur.com/NwynA35.jpg)  
**Answer : C:\RESTRICTED FILES**

## Question 10

What is the Last DHCP IP assigned to this host ?  
Answer in `SYSTEM\ControlSet002\Services\Tcpip\Interfaces`  
![](https://i.imgur.com/iBQPPnP.jpg)  
**Answer : 172.31.2.197**

## Question 11

The suspect seems to have accessed a file containing the secret coffee recipe. What is the name of the file ?  
Answer in `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\pdf`  
![](https://i.imgur.com/jokxCLO.jpg)  
**Answer : secret-recipe.pdf**

## Question 12

The suspect ran multiple commands in the run windows. What command was run to enumerate the network interfaces ?  
Answer in `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU`  
![](https://i.imgur.com/SLhWx5e.jpg)  
**Answer : pnputil /enum-interfaces**

## Question 13

In the file explorer, the user searched for a network utility to transfer files. What is the name of that tool ?  
Answer in `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery`  
![](https://i.imgur.com/FNEdFdD.jpg)  
**Answer : netcat**

## Question 14

What is the recent text file opened by the suspect ?  
Answer in `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`  
![](https://i.imgur.com/x8aDEo5.jpg)  
**Answer : secret-code.txt**

## Question 15

How many times was Powershell executed on this host ?  
Answer in `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count` 
![](https://i.imgur.com/Mstpvv4.jpg)  
**Answer : 3**

## Question 16

The suspect also executed a network monitoring tool. What is the name of the tool ?  
Answer in `SYSTEM\ControlSet001\Sercices\bam\State\UserSettings\S-1-5-21-1966530601-3185510712-10604624-500` 
![](https://i.imgur.com/QlZfFLy.jpg)  
**Answer : wireshark**

## Question 17

Registry Hives also notes the amount of time a process is in focus. Examine the Hives. For how many seconds was ProtonVPN executed ?  
Answer in `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count` 
![](https://i.imgur.com/8crHeTS.jpg)  
**Answer : 343** (5 minutes x 60 + 43)

## Question 18

Everything.exe is a utility used to search for files in a Windows machine. What is the full path from which everything.exe was executed ?  
Answer in `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count` 
![](https://i.imgur.com/VDk6EN9.jpg)  
**Answer : C:\Users\Administrator\Downloads\tools\Everything\Everything.exe**




