## [SSH](https://en.wikipedia.org/wiki/Secure_Shell)
### Authentication Methods
1. Password authentication
2. Public-key authentication
3. Host-based authentication
4. Keyboard authentication
5. Challenge-response authentication
6. GSSAPI authentication

### Default Configuration 
```shell-session
cat /etc/ssh/sshd_config  | grep -v "#" | sed -r '/^\s*$/d'
```
### Dangerous Settings 

|**Setting**|**Description**|
|---|---|
|`PasswordAuthentication yes`|Allows password-based authentication.|
|`PermitEmptyPasswords yes`|Allows the use of empty passwords.|
|`PermitRootLogin yes`|Allows to log in as the root user.|
|`Protocol 1`|Uses an outdated version of encryption.|
|`X11Forwarding yes`|Allows X11 forwarding for GUI applications.|
|`AllowTcpForwarding yes`|Allows forwarding of TCP ports.|
|`PermitTunnel`|Allows tunneling.|
|`DebianBanner yes`|Displays a specific banner when logging in.|

### Footprinting
```shell-session
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py <ip>
```

#### Change Authentication methods 
```shell-session
ssh -v <username>@<ip>
```
```shell-session
ssh -v <username>@<ip> -o PreferredAuthentications=password
```

## Nmap commands
Default nmap scripts
```shell
nmap -p 22 -sC [ip]
```
retrieve version
```shell
nmap -p 22 -sV [ip]
```
Retrieve supported algorithms
```shell
nmap -p 22 --script ssh2-enum-algos [ip]
```
Retrieve weak keys 
```shell
nmap -p 22 --script ssh-hostkeys --script-args ssh_hostkey=full [ip]
```
Check authentication methods
```shell
nmap -p 22 [ip] --script ssh-auth-methods --script-args="ssh.user=root"
```

## Other Commands
Banner Grab 
```shell
nc -vn [ip] 22
```
Metasploit module to enumerate users
```shell
use scanner/ssh/ssh_enumusers
```
