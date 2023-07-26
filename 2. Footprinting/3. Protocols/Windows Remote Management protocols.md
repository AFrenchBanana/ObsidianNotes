## [RDP](https://learn.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol)

Both firewalls must allow RDP. 
Remote computer will need public address if NAT is used. 
Uses [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) since Vista 

### Footprinting 
#### Nmap 
```shell-session
nmap -sV -sC <ip> -p3389 --script rdp*
```
track packages and see cookies?
```shell-session
 nmap -sV -sC <ip> -p3389 --packet-trace --disable-arp-ping -n
```
#### RDP Security Check 
```shell-session
sudo cpan
```

```shell-session
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check

./rdp-sec-check.pl 10.129.201.248
```
#### Create RDP sessions
```shell-session
xfreerdp /u:<user> /p:"<pass>" /v:<ip>
```

## WinRM
Windows Remote Management
Ports 5985 5986
Based on the command line 
Uses simple Object Access Protocol (SOAP)

### Footprinting 
#### Nmap
```shell-session
nmap -sV -sC <ip> -p5985,5986 --disable-arp-ping -n
```

### [Evil-winrm](https://github.com/Hackplayers/evil-winrm)
Test if we can access winrm 
```shell-session
evil-winrm -i <ip> -u <user> -p <pass>
```

### WMI 
Windows Management Instrumentation 
Extension of Common Information Model (CIM)
Port 135

```shell-session
impacket- wmiexec <user>:"<password>"@<ip>"hostname"
```