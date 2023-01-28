<p align="center">
  THM : Dig Dug<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/digdug<br>
  <img src="https://i.imgur.com/7VgBNQR.jpg">
</p>

## Summary

- [Introduction](#introduction)
- [DNS enumeration](#dns-enumeration)
- [Conclusion](#conclusion)

## Introduction

```
Oooh, turns out, this 10.10.154.186 machine is also a DNS server! If we could dig into it, I am sure we could find some interesting 
records! But... it seems weird, this only responds to a special type of request for a givemetheflag.com domain?
```

Let's add this domain to your `/etc/hosts` file :  
```
attacker@AttackBox:~/Dig_Dug$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	AttackBox
10.10.154.186	givemetheflag.com

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

## DNS enumeration

Let's use [dig](https://linux.die.net/man/1/dig) to enumerate the DNS : 
```
attacker@AttackBox:~/Dig_Dug$ dig @10.10.154.186 givemetheflag.com 

; <<>> DiG 9.16.33-Debian <<>> @10.10.154.186 givemetheflag.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50226
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;givemetheflag.com.		IN	A

;; ANSWER SECTION:
givemetheflag.com.	0	IN	TXT	"flag{*******************************}"

;; Query time: 35 msec
;; SERVER: 10.10.154.186#53(10.10.154.186)
;; WHEN: Sat Jan 28 20:20:34 CET 2023
;; MSG SIZE  rcvd: 86
```

And we have the flag !

## Conclusion

This room was made to practice basic DNS enumeration using [dig](https://linux.die.net/man/1/dig). It's a good room for very beginners. Thanks for this room ! And thanks for reading my write ups !
