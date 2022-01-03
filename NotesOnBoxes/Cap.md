# Cap
Box: https://app.hackthebox.com/machines/Cap
## Enumeration
```bash
# Nmap 7.92 scan initiated Sun Jan  2 19:45:26 2022 as: nmap -sC -sV -oA nmap/scan 10.10.10.245
Nmap scan report for 10.10.10.245
Host is up (0.023s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    gunicorn
|_http-server-header: gunicorn
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 NOT FOUND
|     Server: gunicorn
|     Date: Sun, 02 Jan 2022 08:46:45 GMT
|     Connection: close
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 232
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
|     <title>404 Not Found</title>
|     <SNIP...>
|     </body>
|_    </html>
|_http-title: Security Dashboard
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.92%I=7%D=1/2%Time=61D1662E%P=x86_64-pc-linux-gnu%r(GetRe
SF:quest,2FE5,"HTTP/1\.0\<SNIP...>\n");
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jan  2 19:47:36 2022 -- 1 IP address (1 host up) scanned in 130.13 seconds
```
- Notice ftp is open, no anonymous login allowed
- Moving onto http
![[Pasted image 20220102195413.png]]
- Presented with a dashboard and I notice I am already logged in
- Thinking that this is odd, I check stored cookies, nothing there
- Also cannot logout, box must just be designed like this
- Clicking on the options on the left side of the dashboard the IP config and Network status options look interesting
- The machine must be executing ifconfig and netstat respectively to retreive that information
- Can't find a way of poisoning those values to gain RCE
- Clicking on the Seurity Snapshot section, lets me download a pcap file
![[Pasted image 20220103140741.png]]
- Opening the file, looks llike nothing was captured
- Changing the value from `data/1` to `data/0` in the url shows me a page that looks to have data
- Thinking maybe this is an IDOR vulnerability
- Opening this file in wireshak shows data this time
![[Pasted image 20220103141144.png]]
- Clicking on statistics > Conversations to get an idea of who is talking to who, I see there is only one conversation and it is on a 192.168.196.0 network, nothing to do with any of my traffic as I am on a 10.10... network
- Heading over to the TCP tab I notice all but one TCP stream is talking on port 80 and they all seem to containt exactly 10 packets
![[Pasted image 20220103141647.png]]
- This TCP stream that is on a different port and of different length deffinetly stands out
- Following this stream, it looks like someone was trying to log into ftp and we now have a user and their password
![[Pasted image 20220103142022.png]]
- Also looks like nathan tried to access a file notes.txt, that may be interesting
## Some Exploitation
- Using his details to try loging into ftp again was successful
```bash
┌──(tom㉿acer-computer)-[~/htb/cap]
└─$ ftp 10.10.10.245                                                                                    
Connected to 10.10.10.245.
220 (vsFTPd 3.0.3)
Name (10.10.10.245:tom): nathan
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ^C
ftp> 
```
- After switching to binary mode I was able to retreive the user flag
## Stuck on more enumeration
- Trying to find something else to go off i notice the search feature on the dashboard
- I try putting some bad characters in to see if I can get some kind of error
- I was uncuesseful
![[Pasted image 20220103144500.png]]
- I open burp suite and try to see if instead of navigating to `/netstat` i can do something like `/whoami`. That was unsuccessful
- I try going back to the Security Snapshot page to see if I can get any more pcap files... no luck
- I have a feeling that the way forward is to find that notes.txt file, however, I unfortunatly cannot find the transmission of notes.txt anywhere in the network traffic
- **Hint From Walkthrough - ssh using nathans credentials**
## Priv esc
- Was able to ssh into the box as nathan using his ftp credentials
- I run `cat /etc/passwd | grep sh$` to try and identify any other users. Nathan is the only one other than root
- running `sudo -l` shows that nathan is not a part of the sudoers file
- I try reusing nathans password to switch to root, no luck
- Looking at the websites source code, I identify the framework as flask so therefore written in python
- This function stands out to me, initally because of the comment left behind but also the fact that python is executing an os command as uid 0 meaning it is executing as root. Just what I need for privilege escalation
```python
@app.route("/capture")
@limiter.limit("10 per minute")
def capture():

        get_lock()
        pcapid = get_appid()
        increment_appid()
        release_lock()

        path = os.path.join(app.root_path, "upload", str(pcapid) + ".pcap")
        ip = request.remote_addr
        # permissions issues with gunicorn and threads. hacky solution for now.
        #os.setuid(0)
        #command = f"timeout 5 tcpdump -w {path} -i any host {ip}"
        command = f"""python3 -c 'import os; os.setuid(0); os.system("timeout 5 tcpdump -w {path} -i any host {ip}")'"""
        os.system(command)
        #os.setuid(1000)

        return redirect("/data/" + str(pcapid))
```
- I just need to work out how I can inject my own commands into this command
- Unable to work out how to inject my own code I consulted the walkthrough again
- **Walkthough hint - run linpeas**
- Turns out python 3.8 has cap_setuid set, meaning we can just set our uid in python to 0 and then drop to a shell
```python
nathan@cap:~$ python3
Python 3.8.5 (default, Jan 27 2021, 15:41:15) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.setuid(0)
>>> os.system('/bin/bash')
root@cap:~# 
```
- Priv esc turned out to be relatively simple, I definely overthought it

## Review/Take aways
- I should have gone back and reviewed the nmap scan after getting nathans credentials, I perhaps would have then remembered that ssh is running and attempted to log in with his credentials
- After getting stuck, not being able to find a way of poisoning the values to be executed by the server I should have run linpeas instead of consulting the walkthrough for help
- I could have also realised that for a user to use the `setuid` method to run something as root the user either has to be root or have the setuid_cap
