<p align="center">
  THM : Committed<br>
  Difficulty : Easy<br>
  Room link : https://tryhackme.com/room/committed<br>
  <img src="https://i.imgur.com/wXAJ7FG.jpg">
</p>

## Summary

- [Introduction](#introduction)
- [Find the flag](#find-the-flag)
- [Conclusion](#conclusion)

## Introduction

Oh no, not again! One of our developers accidentally committed some sensitive code to our GitHub repository. Well, 
at least, that is what they told us... the problem is, we don't remember what or where! Can you track down what we accidentally committed ?

## Find the flag

First, let's download the `commited.zip` file to our machine (because we will need to download a tool and the target VM doesn't have internet access).
Then we have to extract the zip file `commited.zip` :  
```
┌──(root㉿kali)-[~]
└─# unzip commited.zip 
Archive:  commited.zip
   creating: commited/
   creating: commited/.git/
   creating: commited/.git/logs/
   creating: commited/.git/logs/refs/
   creating: commited/.git/logs/refs/heads/
  inflating: commited/.git/logs/refs/heads/dbint  
  inflating: commited/.git/logs/refs/heads/master  
  inflating: commited/.git/logs/HEAD  
   creating: commited/.git/refs/
   creating: commited/.git/refs/tags/
   creating: commited/.git/refs/heads/
 extracting: commited/.git/refs/heads/dbint  
 extracting: commited/.git/refs/heads/master  
   creating: commited/.git/info/
  inflating: commited/.git/info/exclude  
 extracting: commited/.git/COMMIT_EDITMSG  
   creating: commited/.git/objects/
   creating: commited/.git/objects/da/
 extracting: commited/.git/objects/da/b5a1b99756122e83df66ea1a86d81257b2ec47  
   creating: commited/.git/objects/dc/
 extracting: commited/.git/objects/dc/1ca4ca1d54e7a4ac6757c9b98bd1be0a8ed2f0  
   creating: commited/.git/objects/26/
 extracting: commited/.git/objects/26/bcf1aa99094bf2fb4c9685b528a55838698fbe  
   creating: commited/.git/objects/69/
 extracting: commited/.git/objects/69/d6211898e43bfe15ab5a4cad1690b9be1115f8  
   creating: commited/.git/objects/74/
 extracting: commited/.git/objects/74/2b40ee5d0597b0595f60998305605186ab29db  
   creating: commited/.git/objects/9e/
 extracting: commited/.git/objects/9e/cdc566de145f5c13da74673fa3432773692502  
   creating: commited/.git/objects/info/
   creating: commited/.git/objects/3a/
 extracting: commited/.git/objects/3a/8cc16f919b8ac43651d68dceacbb28ebb9b625  
   creating: commited/.git/objects/b0/
 extracting: commited/.git/objects/b0/eda7db60a1cb0aea86f053816a1bfb7e2d6c67  
   creating: commited/.git/objects/94/
 extracting: commited/.git/objects/94/a7ea670b13f698012abd246ab08b76d95643c8  
   creating: commited/.git/objects/28/
 extracting: commited/.git/objects/28/c36211be8187d4be04530e340206b856198a84  
   creating: commited/.git/objects/c7/
 extracting: commited/.git/objects/c7/18c75179d46ba5a1d21dc351a39c0dfb257d3d  
   creating: commited/.git/objects/08/
 extracting: commited/.git/objects/08/178a40f4b3585566b539985399f51bbcc7ae22  
   creating: commited/.git/objects/6e/
 extracting: commited/.git/objects/6e/1ea88319ae84175bfe953b7791ec695e1ca004  
   creating: commited/.git/objects/0b/
 extracting: commited/.git/objects/0b/8b1d537ea651d504d29c1556d7dcbcf76a5d57  
   creating: commited/.git/objects/pack/
   creating: commited/.git/objects/40/
 extracting: commited/.git/objects/40/754840a68f85ad8d963f1556a3f24b51cef4fa  
   creating: commited/.git/objects/16/
 extracting: commited/.git/objects/16/1979c948240de867b5c0a4079d7e2c7f6d4e04  
   creating: commited/.git/objects/c5/
 extracting: commited/.git/objects/c5/6c470a2a9dfb5cfbd54cd614a9fdb1644412b5  
   creating: commited/.git/objects/38/
 extracting: commited/.git/objects/38/0dd9b32cc8429638c09cc857cc0ef2ff8f8e50  
   creating: commited/.git/objects/df/
 extracting: commited/.git/objects/df/e24c9e9ae78d8339e44d0c9e32dde9b9efe148  
   creating: commited/.git/objects/45/
 extracting: commited/.git/objects/45/b137061d385e2f5c05cccc7fd13873f2ce18b1  
   creating: commited/.git/objects/0e/
 extracting: commited/.git/objects/0e/1d395f33767d795f3ff66ceac6c792629d40d2  
   creating: commited/.git/objects/b3/
 extracting: commited/.git/objects/b3/7056e8583abc13547fe146c7ee9d905ac8488c  
   creating: commited/.git/objects/44/
 extracting: commited/.git/objects/44/f3cb396ce178127b2dca6fa903113152710129  
 extracting: commited/.git/objects/44/7ef7f1f03534fbea17f61ef3c2e610fcf23693  
 extracting: commited/.git/objects/44/1daaaa600aef8021f273c8c66404d5283ed83e  
   creating: commited/.git/objects/54/
 extracting: commited/.git/objects/54/d0271a615735240d22dcd737b4bf26cbe9d43f  
   creating: commited/.git/objects/4e/
 extracting: commited/.git/objects/4e/ca752261f327712539fc04f81e7f335a69b429  
 extracting: commited/.git/objects/4e/16af9349ed8eaa4a29decd82a7f1f9886a32db  
   creating: commited/.git/objects/fd/
 extracting: commited/.git/objects/fd/132b91ad2a752dd39ce22bc1c55c0e5c38ab84  
  inflating: commited/.git/index     
   creating: commited/.git/hooks/
  inflating: commited/.git/hooks/fsmonitor-watchman.sample  
  inflating: commited/.git/hooks/pre-push.sample  
  inflating: commited/.git/hooks/pre-merge-commit.sample  
  inflating: commited/.git/hooks/pre-rebase.sample  
  inflating: commited/.git/hooks/post-update.sample  
  inflating: commited/.git/hooks/prepare-commit-msg.sample  
  inflating: commited/.git/hooks/update.sample  
  inflating: commited/.git/hooks/commit-msg.sample  
  inflating: commited/.git/hooks/applypatch-msg.sample  
  inflating: commited/.git/hooks/pre-applypatch.sample  
  inflating: commited/.git/hooks/pre-commit.sample  
  inflating: commited/.git/hooks/pre-receive.sample  
  inflating: commited/.git/config    
 extracting: commited/.git/HEAD      
   creating: commited/.git/branches/
  inflating: commited/.git/description  
  inflating: commited/main.py        
  inflating: commited/Readme.md
```

Now, we need to download [GitTools](https://github.com/internetwache/GitTools) :  
```
┌──(root㉿kali)-[~]
└─# wget https://github.com/internetwache/GitTools/releases/download/v0.0.1/gitTools-v0.0.1.zip
--2023-03-14 10:32:26--  https://github.com/internetwache/GitTools/releases/download/v0.0.1/gitTools-v0.0.1.zip
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/34182773/9e4a9e00-addd-11eb-93dd-036c3ec56ae9?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230314%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230314T103157Z&X-Amz-Expires=300&X-Amz-Signature=175959b12a2837be4614527d49ede350b7b164d968e919cf5b1be97b51f0f067&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=34182773&response-content-disposition=attachment%3B%20filename%3DgitTools-v0.0.1.zip&response-content-type=application%2Foctet-stream [following]
--2023-03-14 10:32:27--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/34182773/9e4a9e00-addd-11eb-93dd-036c3ec56ae9?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230314%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230314T103157Z&X-Amz-Expires=300&X-Amz-Signature=175959b12a2837be4614527d49ede350b7b164d968e919cf5b1be97b51f0f067&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=34182773&response-content-disposition=attachment%3B%20filename%3DgitTools-v0.0.1.zip&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.111.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 219319 (214K) [application/octet-stream]
Saving to: ‘gitTools-v0.0.1.zip’

gitTools-v0.0.1.zip               100%[===========================================================>] 214.18K  --.-KB/s    in 0.03s   

2023-03-14 10:32:27 (7.16 MB/s) - ‘gitTools-v0.0.1.zip’ saved [219319/219319]

┌──(root㉿kali)-[~]
└─# unzip gitTools-v0.0.1.zip -d GitTools
Archive:  gitTools-v0.0.1.zip
   creating: GitTools/.git/
   creating: GitTools/.git/branches/
   creating: GitTools/.git/hooks/
  inflating: GitTools/.git/hooks/applypatch-msg.sample  
  inflating: GitTools/.git/hooks/commit-msg.sample  
  inflating: GitTools/.git/hooks/post-update.sample  
  inflating: GitTools/.git/hooks/pre-applypatch.sample  
  inflating: GitTools/.git/hooks/pre-commit.sample  
  inflating: GitTools/.git/hooks/pre-push.sample  
  inflating: GitTools/.git/hooks/pre-rebase.sample  
  inflating: GitTools/.git/hooks/prepare-commit-msg.sample  
  inflating: GitTools/.git/hooks/update.sample  
   creating: GitTools/.git/info/
  ...
  ...
```

Now, we can use `Extractor` to extract commits for the repository :  
```
┌──(root㉿kali)-[~]
└─# ./GitTools/Extractor/extractor.sh ./commited ./dump
###########
# Extractor is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########
[*] Destination folder does not exist
[*] Creating...
[+] Found commit: c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5
[+] Found file: /root/./dump/0-c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5/Note
[+] Found file: /root/./dump/0-c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5/Readme.md
[+] Found file: /root/./dump/0-c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5/main.py
[+] Found commit: 9ecdc566de145f5c13da74673fa3432773692502
[+] Found file: /root/./dump/1-9ecdc566de145f5c13da74673fa3432773692502/Readme.md
[+] Found file: /root/./dump/1-9ecdc566de145f5c13da74673fa3432773692502/main.py
[+] Found commit: b0eda7db60a1cb0aea86f053816a1bfb7e2d6c67
[+] Found file: /root/./dump/2-b0eda7db60a1cb0aea86f053816a1bfb7e2d6c67/Readme.md
[+] Found file: /root/./dump/2-b0eda7db60a1cb0aea86f053816a1bfb7e2d6c67/main.py
[+] Found commit: 441daaaa600aef8021f273c8c66404d5283ed83e
[+] Found file: /root/./dump/3-441daaaa600aef8021f273c8c66404d5283ed83e/Readme.md
[+] Found file: /root/./dump/3-441daaaa600aef8021f273c8c66404d5283ed83e/main.py
[+] Found commit: 4e16af9349ed8eaa4a29decd82a7f1f9886a32db
[+] Found file: /root/./dump/4-4e16af9349ed8eaa4a29decd82a7f1f9886a32db/Note
[+] Found file: /root/./dump/4-4e16af9349ed8eaa4a29decd82a7f1f9886a32db/Readme.md
[+] Found file: /root/./dump/4-4e16af9349ed8eaa4a29decd82a7f1f9886a32db/main.py
[+] Found commit: 26bcf1aa99094bf2fb4c9685b528a55838698fbe
[+] Found file: /root/./dump/5-26bcf1aa99094bf2fb4c9685b528a55838698fbe/Readme.md
[+] Found file: /root/./dump/5-26bcf1aa99094bf2fb4c9685b528a55838698fbe/main.py
[+] Found commit: 6e1ea88319ae84175bfe953b7791ec695e1ca004
[+] Found file: /root/./dump/6-6e1ea88319ae84175bfe953b7791ec695e1ca004/Note
[+] Found file: /root/./dump/6-6e1ea88319ae84175bfe953b7791ec695e1ca004/Readme.md
[+] Found file: /root/./dump/6-6e1ea88319ae84175bfe953b7791ec695e1ca004/main.py
[+] Found commit: 3a8cc16f919b8ac43651d68dceacbb28ebb9b625
[+] Found file: /root/./dump/7-3a8cc16f919b8ac43651d68dceacbb28ebb9b625/Note
[+] Found file: /root/./dump/7-3a8cc16f919b8ac43651d68dceacbb28ebb9b625/Readme.md
[+] Found file: /root/./dump/7-3a8cc16f919b8ac43651d68dceacbb28ebb9b625/main.py
[+] Found commit: 28c36211be8187d4be04530e340206b856198a84
[+] Found file: /root/./dump/8-28c36211be8187d4be04530e340206b856198a84/Readme.md
[+] Found file: /root/./dump/8-28c36211be8187d4be04530e340206b856198a84/main.py
```

Finally, let's find the flag :  
```
┌──(root㉿kali)-[~/dump]
└─# grep -iR flag
7-3a8cc16f919b8ac43651d68dceacbb28ebb9b625/main.py:    password="flag{********************************}" # Password Goes Here
7-3a8cc16f919b8ac43651d68dceacbb28ebb9b625/main.py:    password="flag{********************************}", #password Goes here
7-3a8cc16f919b8ac43651d68dceacbb28ebb9b625/main.py:    password="flag{********************************}",
```

## Conclusion

This room is useful to practice the usage of the `Extractor` from [GitTools](https://github.com/internetwache/GitTools) ! Thanks for this room, and thanks for 
reading my write ups !
