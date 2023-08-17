## #SSH for windows:
### [plink.exe](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
Short for PuTTY Link 
Windows command-line SSH tool. 
Can be used to create dynamic port forwards and SOCKS proxies. 
Windows did not have SSH client before 2018.
![[Pivotting RDP.png]]
```cmd
plink -D 9050 user@target
```
### Proxifier 
Can start SOCKS tunnels via SSH.
![[Proxifier.png]]

## SSH #Pivoting with [Sshuttle](https://github.com/sshuttle/sshuttle)
Removes the need to configure proxychains 
Onlt works when pivoting over SSH and not TOR or HTTPS.

```shell
sudo apt-get install sshuttle
```

```shell
sudo sshuttle -r <user>@<pivotIP> <target IP or network> -v 
```
Creates an iptables to redirect all traffic to target IP 
Can use any tool without proxychains
## Web Server Pivoting with [Rpivot](https://github.com/klsecservices/rpivot)
Reverse SOCKS proxy tool.
![[Pivot 3.png]]
```shell
sudo git clone https://github.com/klsecservices/rpivot.git
```

```shell
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```
Transfer rpvitot to the target 
```shell
scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/
```
Run client on Pivot 
```shell
 python2.7 client.py --server-ip 10.10.14.18 --server-port 9999
```
Configure proxy chains to pivot over local server 
```shell
proxychains firefox-esr 172.16.5.135:80
```
Connecting to a web server using HTTP-Proxy & NTLM Auth
```shell
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```

## Port Forwarding with Windows [Netsh](https://learn.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts)
#Netsh uses:
- `Finding routes`
- `Viewing the firewall configuration`
- `Adding proxies`
- `Creating port forwarding rules`
![[Port Forward.png]]
```cmd
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=<localip on pivot> connectport=3389 connectaddress=<targetIP>
```

```cmd
netsh.exe interface portproxy show v4tov4
```
Connect to #RDP over port 8080 to the pivot machine. 
