# Blue
Box: https://app.hackthebox.com/machines/Blue
## Enumeration
- Nmap scan
```bash
# Nmap 7.92 scan initiated Sun Jan  2 16:27:39 2022 as: nmap -sC -sV -oA nmap/scan -T 4 -v 10.10.10.40
Nmap scan report for 10.10.10.40
Host is up (0.027s latency).
Not shown: 991 closed tcp ports (conn-refused)
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2022-01-02T05:29:52
|_  start_date: 2022-01-02T05:27:32
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-01-02T05:29:50+00:00
|_clock-skew: mean: 1m07s, deviation: 0s, median: 1m07s

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jan  2 16:28:51 2022 -- 1 IP address (1 host up) scanned in 72.28 seconds
```
- Starting at the top, did not really know what to do with msrpc on port 135
- Nmap script identified a user as haris
### Enumeration of smb
- Seeing that smb is running attempted to list shares without auth
![[Pasted image 20220102164835.png]]
- Dont have access to ADMIN\$ or C\$ shares
- IPC$ gives error
- Enumerating the Share and Users shares doesnt give up anything
- Tried running [crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec) to see what version of smb it is running 
- cme shows that it is smb v1
![[Pasted image 20220102164316.png]]
- Thinking is it vulnerable to the eternal blue exploit
- Used metasploits smb scanner
![[Pasted image 20220102170410.png]]
- Apparently it is vulnerable to eternalblue
### Exploitation
- Using the eternalblue expoit in metasploit gave me a shell
![[Pasted image 20220102170832.png]]
- Looking at the users, looks like haris is the only one
![[Pasted image 20220102171043.png]]
- Got the user flag on her desktop
#### Priv esc
- Metasploits suggester has some suggestions of exploits to run
![[Pasted image 20220102171436.png]]
- After attempting  several of these suggestions I realised that the eternal blue exploit I ran gave me access as user `nt authority\system` and therefore we are already admin
- Got root flag on the desktop
## Learning
- I should have checked who I was as the first thing when I got the shell
- Instead of manually enumerating the shares to workout the permissions with `smbclient` I could have used `smbmap` to show me the permissions