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

## [RSYNC](https://linux.die.net/man/1/rsync)

#Port 873 

Tool for copying files locally and remotely
 
[Potential abuse methods](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)

#### Scan For RSYNC 
```shell-session
sudo nmap -sV -p 873 127.0.0.1
```

#### Accessible Shares
```shell-session
nc -nv 127.0.0.1 873
```

#### Enumerate open shares
```shell-session
rsync -av --list-only rsync://127.0.0.1/dev
```

#### Sync Files 
```shell-session 
rsync -av rsync://127.0.0.1/dev
```

## R Services 

Remote access services/ commands between unix hosts.

Ports 512, 513, 514 

[R-commands](https://en.wikipedia.org/wiki/Berkeley_r-commands)

|**Command**|**Service Daemon**|**Port**|**Transport Protocol**|**Description**|
|---|---|---|---|---|
|`rcp`|`rshd`|514|TCP|Copy a file or directory bidirectionally from the local system to the remote system (or vice versa) or from one remote system to another. It works like the `cp` command on Linux but provides `no warning to the user for overwriting existing files on a system`.|
|`rsh`|`rshd`|514|TCP|Opens a shell on a remote machine without a login procedure. Relies upon the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files for validation.|
|`rexec`|`rexecd`|512|TCP|Enables a user to run shell commands on a remote machine. Requires authentication through the use of a `username` and `password` through an unencrypted network socket. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.|
|`rlogin`|`rlogind`|513|TCP|Enables a user to log in to a remote host over the network. It works similarly to `telnet` but can only connect to Unix-like hosts. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.|

#### Trusted hosts
Locatd in /etc/hosts.equiv
```shell-session
cat /etc/hosts.equiv
```

#### Nmap 
```shell-session
sudo nmap -sV -p 512,513,514 <ip>
```

#### Logging in 
```shell-session
rlogin <ip>-l <user>
```

#### List authenticated users 
```shell-session
rwho
```
```shell-session
rusers -al <ip>
```
