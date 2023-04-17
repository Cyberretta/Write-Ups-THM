<p align="center">
  THM : tomghost<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/tomghost<br>
  <img src="https://i.imgur.com/8yS9vFp.jpg">
</p>

## Summary

- [Nmap scan](#nmap-scan)
- [Ghostcat exploitation](#ghostcat-exploitation)
- [Privilege escalation (merlin)](#privilege-escalation-merlin)
- [Privilege escalation (root)](#privilege-escalation-root)
- [Conclusion](#conclusion)

## Nmap scan

Let's run an agressive [nmap](https://nmap.org/book/man.html) scan against the target :  
![](https://i.imgur.com/b7rbsKs.jpg)  

## Ghostcat exploitation

Since we know Tomcat 9.0.30 is running on the target, we can look for exploits on [Exploit-DB](https://www.exploit-db.com/) :  
![](https://i.imgur.com/WdsPxMj.jpg)  

This exploit should work on Tomcat 9.0.30. Let's run the [Metasploit Framework](https://www.metasploit.com/) console using `msfconsole` and type `search Ghostcat` :  
![](https://i.imgur.com/NOoNTEm.jpg)  

Let's use `use 0` to select this exploit and then `options` to see the options for this exploit :  
![](https://i.imgur.com/BUhbGqn.jpg)  

Here, we only need to set the RHOSTS value to the target IP address using `set RHOSTS [TARGET_IP]`. After that, we can use `run` to run the exploit :  
![](https://i.imgur.com/l1MiA20.jpg)  

We found a set of credentials. Let's try to use it on port 22 (SSH) :  
![](https://i.imgur.com/DDoMBu2.jpg)  

We are logged in as `skyfuck` !

## Privilege escalation (merlin)

Let's take a look at the files in our home directory :  
![](https://i.imgur.com/ek4U1VN.jpg)  

Let's download the files `tryhackme.asc` and `credentials.pgp` to our attacking host using netcat. Then, we can extract a hash from `tryhackme.asc` using `gpg2john` :  
![](https://i.imgur.com/BCK3uwv.jpg)  

Now, let's try to crack this hash using [john](https://www.openwall.com/john/doc/) :  
![](https://i.imgur.com/GTJ7A0c.jpg)  

Now that we have the passphrase, we can decrypt the file `credentials.pgp` and read its content :  
![](https://i.imgur.com/JpickdO.jpg)  

Let's try to use those crendentials to login as `merlin` :  
![](https://i.imgur.com/0lB3R4b.jpg)  

Now we are logged in as `merlin` !

## Privilege escalation (root)

We can get the first flag in `/home/merlin/user.txt` :  
![](https://i.imgur.com/rSbMVkT.jpg)  

Let's take a look at our sudo rights using `sudo -l` :  
![](https://i.imgur.com/OQzcct2.jpg)  

We can run `zip` as `root` without password. We can search for `zip` on [GTFOBins](https://gtfobins.github.io/) :  
![](https://i.imgur.com/jZTRsBl.jpg)  

Let's get a shell using this commands :  
![](https://i.imgur.com/EKmeR65.jpg)  

And now we are root ! Let's get the root flag in `/root/root.txt` :  
![](https://i.imgur.com/BZ1Vwf2.jpg)  

## Conclusion

In this room, we practiced :  
- Services and open ports enumeration using [nmap](https://nmap.org/book/man.html)
- Ghostcat exploitation on Tomcat 9.0.30
- Linux enumeration
- Cracking .asc passphrase
- Decrypting a .pgp file
- Linux privilege escalation using sudo rights on zip

Thanks for this room and thanks for reading my write ups !
