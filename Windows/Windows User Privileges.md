
Authority Levels:
	1. NT Authority\ System 
	2. administrator 
	3. Administrators 
	4. non-privileged domain user part of admin group.
	5. Domain user part of a Administrator group.

Authorisation Process 
	Every Security Identifier is identified by a unique Security Identifier (SID)
		https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers
	Windows authorisation and access control process
	 Users access tokens are compared against Access Control Entries (ACE)
		 https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-entries
	 ![[Windows Authorisation Process.png]]
		 https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-principals

Group/ User Rights and Privileges 
	Groups
		Default Administrators 
			Domain Admins and Enterprise Admins are "Super Groups"
		Server Operators 
			Members can modify services, access SMB shares and backup files
		Backup Operators
			Members are allowed to log onto DCs locally and should be considered Domain Admins. They can make shadow copies of the SAM/NTDS database, read the registry remotely, and access the file system on the DC via SMB. This group is sometimes added to the local Backup Operators group on non-DCs.
		Print Operators
			Members can logon to DCs locally and "trick" windows into loading a malicious driver. 
		Hyper-V Administrators
			If a Virtual DCs is present these should be considered domain admins
		Account Operators
			Modify non-protected accounts and groups in the domain
		Remote Desktop Users
			Not given any useful permission by default but often given rights such as: "Alow Login through RDP" and can move laterally through RDP
		Remote Management Users 
			Can log on to DCs with PSremoting
		Group Policy Creator Owners
			can create new GPOs but would never be delegated permissions to link GPOs to a container like a DC
		Schema Admins
			Members can modify the Active Directory schema structure and backdoor any-to-be created Group/GPO by adding a compromised account to the default object ACL.
		DNS Admins
			Can load DLL in a DC. Cannot restart the DNS server. Load a malicious DLL and wait for reboot as a persistence mechanism.  
			May often result in a crash. better to exploit a WDAP record. 
				https://cube0x0.github.io/Pocing-Beyond-DA/
	Users
		https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-rights-assignment
		List user assigned rights - whoami /priv
	Standard rights 
		Administrator	```
			SeIncreaseQuotaPrivilege
			SeSecurityPrivilege
			SeTakeOwnershipPrivilege
			SeLoadDriverPrivilege
			SeSystemProfilePrivilege
			SeSystemtimePrivilege
			SeProfileSingleProcessPrivilege
			SeIncreaseBasePriorityPrivilege
			SeCreatePagefilePrivilege
			SeBackupPrivilege
			SeRestorePrivilege
			SeShutdownPrivilege 
			SeDebugPrivilege    
			SeSystemEnvironmentPrivilege
			SeChangeNotifyPrivilege
			SeRemoteShutdownPrivilege 
			SeUndockPrivilege
			SeManageVolumePrivilege
			SeImpersonatePrivilege
			SeCreateGlobalPrivilege
			SeIncreaseWorkingSetPrivilege
			SeTimeZonePrivilege
			SeCreateSymbolicLinkPrivilege
			SeDelegateSessionUserImpersonatePrivilege
			```
		User  ```
			SeChangeNotifyPrivilege
			SeIncreaseWorkingSetPrivilege
			```
		Backup Operator ```
			SeShutdownPrivilege 
			SeChangeNotifiyPrivilege
			SeIncreaseWorkingSePrivilege
			```
		