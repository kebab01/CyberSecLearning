# Blocky
Box: https://app.hackthebox.com/machines/Blocky
## Enumeration
- Nmap scan
```bash
# Nmap 7.92 scan initiated Tue Jan  4 13:30:48 2022 as: nmap -sC -sV -oA nmap/scan -T 4 -v 10.10.10.37
Nmap scan report for 10.10.10.37
Host is up (0.019s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp closed sophos
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jan  4 13:31:02 2022 -- 1 IP address (1 host up) scanned in 13.92 seconds
```
- First thing to check is if anonymous login is allowed, it is unlickely since nmap did not pick that up with the script scan but worth trying just incase
- Uncussessful with ftp, now to checking out the website
![[Pasted image 20220104133345.png]]
- The webpage looks like a pretty standard wordpress page
- Since I know its wordpress Ill try finding the admin login and try some default creds
- Unlucky with default credentails, lets try and enumerate users
- On the main page I find a blog post by the author notch
- Going to the url `http://10.10.10.37/?author=1` confirms that he is an author
- I go back to the admin login page and try to log in as notch with some default credentials, however i was unlucky
- On the main page there were links to downloading two xml documents "Entries RSS" and "Comments RSS"
- Examining these documents I found that the posts are reference as `/?p=<num>`
- The post by notch was p=5
- I try enumerating other numers of p. Nothing comes up other than an exmaple wordpress blog with little information
- As I feel I have done as much manual enumeration as possible I run wp-scan to try and identify some vulnerabilities
- Whilst wp-scan is running I also decide to run an all ports scan with nmap to try and give me more to go off
- wp-scan identifies lots of vulnerabilites, but looks like notch is the only user
- With most of the vulnerabilites be XSS related I can really make the most of them as there are no other users to execute the XSS
- Nmap all port scan finished and identifies a minecraft server on port 25565
- According to wpscan XML-RPC is enabled, this means a brute force attack could be relatively quick, ill run notch against the rockyou.txt list
- With all of my leads exausted I run gobuster to try and identify more files on the server
- Gobuster identifies a phpmyadmin login page
![[Pasted image 20220104141831.png]]
- Attempted some default credentials with no luck
- **Walkthrough hint /plugins**
- I unfortunatley confused the /plugins page with worpresses wp-content/plugins and therefore did not originally pay attention to that dirbuster result
- The `/plugins` page reveals two .jar files we can download
- After doing some googling on how to decompile java files I found a program call jd-gui
- Opening up this file in jd-gui it reveals some sql credentials
```java
package com.myfirstplugin;  
  
public class BlockyCore {  
 public String sqlHost = "localhost";  
   
 public String sqlUser = "root";  
   
 public String sqlPass = "8YsqfCTnvxAUeduzjNSXe22";  
   
 public void onServerStart() {}  
   
 public void onServerStop() {}  
   
 public void onPlayerJoin() {  
 sendMessage("TODO get username", "Welcome to the BlockyCraft!!!!!!!");  
 }  
   
 public void sendMessage(String username, String message) {}  
}
```
## Exploitation
- I attempt to log into ssh as notch and try reusing that password
- I am successful
- I run `sudo -l` to see what I can do
```bash
notch@Blocky:~$ sudo -l
[sudo] password for notch: 
Matching Defaults entries for notch on Blocky:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User notch may run the following commands on Blocky:
    (ALL : ALL) ALL
notch@Blocky:~$ 
```
## Priv esc
- Looks like I have full sudo privileges, because I know notches password it is an easy priv esc
- `sudo su` and now I am root
## Learning/ Reflection
- I firstly should have started a gobuster scan before going down the rabit hole of trying to find other blog posts, and trying to research wordpress vulnerabilites
- If I had realised that the `/plugins` page was not the `/wp-content/plugins` I would not have needed to consult the walkthrough for a hint
- Once I was able to find those .jar files, the box was relatively simple form there
- Main takeaway would be always enumerate fully and dont disregard a result without investigating it first