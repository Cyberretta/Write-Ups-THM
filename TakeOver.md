<p align="center">
  THM : TakeOver<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/takeover<br>
  <img src="https://i.imgur.com/aK2ENq7.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Finding the first subdomain](#finding-the-first-subdomain)
- [Finding the second subdomain](#finding-the-second-subdomain)
- [Conclusion](#conclusion)

## Nmap scan

Let's run a basic nmap scan against the target (we don't need to make an aggressive scan since we just have to find the subdomain that can be taken over, so we just need
to know on what ports is the webserver) :  
```
attacker@AttackBox:~/TakeOver$ nmap futurevera.thm -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-21 15:52 CET
Nmap scan report for futurevera.thm (10.10.125.247)
Host is up (0.047s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 0.79 seconds
```

## Finding the first subdomain

In the statement of the room, we are told the company is rebuilding their `support`. They may have a subdomain named `support`. Let's add it to `/etc/hosts`, and see 
what we can find on this subdomain :  
```
attacker@AttackBox:~/TakeOver$ sudo nano /etc/hosts
attacker@AttackBox:~/TakeOver$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	AttackBox
10.10.125.247	futurevera.thm	support.futurevera.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now, let's see what we can find on this subdomain using a web browser :  
![](https://i.imgur.com/2wnlJzF.jpg)  

Let's click on `Show certificate` (`Afficher le certificat` in french) :  
![](https://i.imgur.com/J9P9IiH.jpg)  

## Finding the second subdomain

If we go a little lower on the certificate, we can find an alternative subdomain :  
![](https://i.imgur.com/ZoB8qIM.jpg)  

Let's add it to our `/etc/hosts` :  
```
attacker@AttackBox:~/TakeOver$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	AttackBox
10.10.125.247	futurevera.thm	support.futurevera.thm	secret************.support.futurevera.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

And now, let's see what we can find on it using a web browser (you need to access this subdomain on port 80, not port 443) :  
![](https://i.imgur.com/dZtlwN8.jpg)  

We are redirected to a AWS web server, but not found. And we can see the flag in the URL ! 
So we found the subdomain the black hats can take over (`flag{*******************************}.s3-website-us-west-3.amazonaws.com`) and the flag !

## Conclusion

This room was very interesting, it made me search and try to understand what is `Subdomain takeover`, so I learned something new with this room.  
If you want to learn more about `Subdomain takeover`, here is some useful links :  
- https://book.hacktricks.xyz/pentesting-web/domain-subdomain-takeover
- https://www.hackerone.com/application-security/guide-subdomain-takeovers
- https://developer.mozilla.org/en-US/docs/Web/Security/Subdomain_takeovers

Thanks for this room ! And thanks for reading my writeups !
