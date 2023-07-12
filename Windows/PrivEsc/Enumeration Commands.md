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

	  System infomation 
		'systeminfo' or '[wmic] qfe' or 'Get-Hotfix'

	Work out when a system was patched:
			https://www.catalog.update.microsoft.com/Search.aspx?q=hotfix

	Installed Programs 
		'wmic product get name' or 'Get-WmiObject -Class Win32_Product | select Name, Version' 
		[WMIC](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page)
	Running Processes = 'netstat -ano'
		 Look for active network connections on loopback
			 127.0.0.1 / ::1 

	Users Enumiration
		Logged in users: 'query user'
		whoami
		Privelages = 'whoami \priv'
		User Groups = 'whoami \groups'
		All users = 'net user'
		All Groups = 'net localgroup'
		Group Details 'net localgroup "<group name>"'
		Password Policy = 'net accounts'

	Name Pipes
		Files stored in memory
				Can communicate over half-duplex or duplex
		Use tool called PipeList
			List Pipes 
				cmd = 'pipelist.exe / accepteula'
				powershell = "gci \\.\pipe\"
			Review permissions of named pipes:
				powershell = "`.\accesschk.exe /accepteula \pipe\*`."
			Attacking a Pipe
				List Pipes that have write permissions:
					"accesschk.exe -accepteula -w \pipe\* -v"
		https://learn.microsoft.com/en-us/sysinternals/downloads/pipelist	