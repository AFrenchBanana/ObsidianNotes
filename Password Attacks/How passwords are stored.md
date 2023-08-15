## Linux 
### /etc/shadow
|   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|
|htb-student:|`$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:`|18955:|0:|99999:|7:|:|:|:|
|`<username>`:|`<encrypted password>`:|`<day of last change>`:|`<min age>`:|`<max age>`:|`<warning period>`:|`<inactivity period>`:|`<expiration date>`:|`<reserved field>`|

|   |   |   |
|---|---|---|
|`$ <id>`|`$ <salt>`|`$ <hashed>`|
|`$ y`|`$ j9T`|`$ 3QSBB6CbHEu...SNIP...f8Ms`|

|**ID**|**Cryptographic Hash Algorithm**|
|---|---|
|`$1$`|[MD5](https://en.wikipedia.org/wiki/MD5)|
|`$2a$`|[Blowfish](https://en.wikipedia.org/wiki/Blowfish_(cipher))|
|`$5$`|[SHA-256](https://en.wikipedia.org/wiki/SHA-2)|
|`$6$`|[SHA-512](https://en.wikipedia.org/wiki/SHA-2)|
|`$sha1$`|[SHA1crypt](https://en.wikipedia.org/wiki/SHA-1)|
|`$y$`|[Yescrypt](https://github.com/openwall/yescrypt)|
|`$gy$`|[Gost-yescrypt](https://www.openwall.com/lists/yescrypt/2019/06/30/1)|
|`$7$`|[Scrypt](https://en.wikipedia.org/wiki/Scrypt)|

### /etc/passwd

|   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|
|`htb-student:`|`x:`|`1000:`|`1000:`|`,,,:`|`/home/htb-student:`|`/bin/bash`|
|`<username>:`|`<password>:`|`<uid>:`|`<gid>:`|`<comment>:`|`<home directory>:`|`<cmd executed after logging in>`|

x indicated encrypted password is in /etc/shadow
If empty may be able to login without a password
## Windows 
![[Pasted image 20230801103041.png]]
#LSASS #WinLogin #SAM
Local interactive logon performed between = **WinLogon** , **LoginUI**, **credential provider**, **LSASS**
**Authorisation Packages**, **SAM/AD** 

#### [Winlogon](https://www.microsoftpressstore.com/articles/article.aspx?p=2228450&seqNum=8)
Trusted process responsible for managing security-related user interactions.
	Launching LogonUI to enter passwords
	Changing passwords 
	Locking and unlocking workstation
Relies on Credential providers ccalled COM objects in DLLs.

#### [LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service)

[Local security Authority Subsystem Service](<https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc961760(v=technet.10)?redirectedfrom=MSDN>)

Collection of Modules and has access to all authentication process found in `System32\Lsass.exe`.
Responsible for local system security policy, user authentication and sending audit logs to event logs. 

|**Authentication Packages**|**Description**|
|---|---|
|`Lsasrv.dll`|The LSA Server service both enforces security policies and acts as the security package manager for the LSA. The LSA contains the Negotiate function, which selects either the NTLM or Kerberos protocol after determining which protocol is to be successful.|
|`Msv1_0.dll`|Authentication package for local machine logons that don't require custom authentication.|
|`Samsrv.dll`|The Security Accounts Manager (SAM) stores local security accounts, enforces locally stored policies, and supports APIs.|
|`Kerberos.dll`|Security package loaded by the LSA for Kerberos-based authentication on a machine.|
|`Netlogon.dll`|Network-based logon service.|
|`Ntdsa.dll`|This library is used to create new records and folders in the Windows registry.|



#### [SAM](<https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc756748(v=ws.10)?redirectedfrom=MSDN>)
Database file that stores local users passwords. 
`System32\config\SAM` mounted onto `HKLM\SAM`
SYSKEY (syskey.exe) encrypts hard disk copy to make offline cracking harder.

#### Credential Manager
![[Pasted image 20230801104051.png]]
Feature allows users to save credentials used access network resources and websites. 
Encrypted and stored in:
```powershell
C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```
#NTDS
#### NTDS
Logon requests get sent to NTDS.dit on the DC. [Except for Read-Only DCs.](https://learn.microsoft.com/en-us/windows/win32/ad/rodc-and-active-directory-schema)

Database File storing items such as
User accounts: Username and password 
group Accounts 
Computer Accounts 
GPO 
