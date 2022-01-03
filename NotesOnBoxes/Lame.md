# Lame
Box: https://app.hackthebox.com/machines/Lame
## Enumeration
- Running normal nmap scan `nmap -sC -sV -oA nmap/scan -T 4 10.10.10.3 -v` did not work as I got a host down error
- Tried running again with `-Pn` to treat all hosts as up
```bash
# Nmap 7.92 scan initiated Sun Jan  2 18:13:11 2022 as: nmap -sC -sV -oA nmap/scan -T 4 -v -Pn 10.10.10.3
Nmap scan report for 10.10.10.3
Host is up (0.021s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.22
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2022-01-02T02:13:28-05:00
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 2h29m57s, deviation: 3h32m11s, median: -5s

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jan  2 18:14:08 2022 -- 1 IP address (1 host up) scanned in 56.63 seconds
```
- Starting at the top i see that anonymous ftp login is allowe
- Tried logging in
![[Pasted image 20220102181657.png]]
- ftp doesnt seem to have anything
- smb is running version 3.0.20-Debian
- Know that this is a debian box (ssh also told me that)
- With not much else to go off, I try doing a search with searchsploit
```bash
┌──(tom㉿acer-computer)-[~/htb/lame]
└─$ searchsploit samba 3.0
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                  |  Path
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Samba 3.0.10 (OSX) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                                                                                                            | osx/remote/16875.rb
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                                                                                                                          | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                                                                                | unix/remote/16320.rb
Samba 3.0.21 < 3.0.24 - LSA trans names Heap Overflow (Metasploit)                                                                                                              | linux/remote/9950.rb
Samba 3.0.24 (Linux) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                                                                                                          | linux/remote/16859.rb
Samba 3.0.24 (Solaris) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                                                                                                        | solaris/remote/16329.rb
Samba 3.0.27a - 'send_mailslot()' Remote Buffer Overflow                                                                                                                        | linux/dos/4732.c
Samba 3.0.29 (Client) - 'receive_smb_raw()' Buffer Overflow (PoC)                                                                                                               | multiple/dos/5712.pl
Samba 3.0.4 - SWAT Authorisation Buffer Overflow                                                                                                                                | linux/remote/364.pl
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                           | linux/remote/7701.txt
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                           | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                                                                                   | linux_x86/dos/36741.py
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
- The third result looked interesting
- Searching for that exploit in msfconsole shows 1 result
## Exploitation
- After configuring the exploit and runing it, meterpreter gave me a shell
- Running `whoami` shows that I am now root, no need for priv esc
- Trying to spawn a proper tty crashed my shell