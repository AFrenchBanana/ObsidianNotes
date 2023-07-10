Enumeration:

	RDP Via Linux - rdesktop <ip> -u <username> -p <password>

	print routes = route print

	Windows Defender Status = Get-MpComputerStatus

	Applocker Rules = Get-AppLockerPolicy -Effective | select - ExpandProperty RuleCollections

	Task List = tasklist \svc
	Common Windows Processes:
			Session manager - smss.exe
			Client Server Runtime Subsystem - csrss.exe
			WinLogon - winlogin.exe
			Local Security Authority Subsystem service - LSASS)
			Service Host - svchost.exe
	Enviromental Variables: - Check with 'set'
		Python or Java to allow execution of these files placed in $PATH
		If $PATH is writeable - DLL Injection?
		Windows looks for CWD then $PATH
	System info - 'systeminfo' or '[wmic] qfe' or 'Get-Hotfix'
	Work out when a system was patched:
			https://www.catalog.update.microsoft.com/Search.aspx?q=hotfix
	Installed Programs 'wmic product get name' or 'Get-WmiObject -Class Win32_Product | select Name, Version' 
		[WMIC](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page)
	Running Processes = 'netstat -ano' 	
	Logged in users: 'query user'
	Users
		whoami
		Privelages = 'whoami \priv'
		User Groups = 'whoami \groups'
		All users = 'net user'
		All Groups = 'net localgroup'
		Group Details 'net localgroup "<group name>"'
		Password Policy = 'net accounts'
	
	

		
