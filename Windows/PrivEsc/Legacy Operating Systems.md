EOL Dates
	```	Windows XP | April 8, 2014 
	Windows Vista | April 11 2017
	Windows 7 | January 14 2020
	Windows 8 | January 12 2016 
	Windows 8.1  January 10 2013
	Windows 10 release 1507 | May 9 2017 
	Windows 10 release 1703 | October 9 2018 
	Windows 10 release 1809 | November 10 2020 
	Windows 10 release 1903 | December 8 2020 
	Windows 10 release 2004 | December 14 2021
	Windows 10 release 20H2 | May 10 20202
	Windows Server 2003 | April 8 2014
	Windows Server 2003 R2 | July 14 2015
	Windows Server 2008 | January 14 2020
	Windows Server 2008 R2 | January 14 2020 
	Windows Server 2012 | October 10 2023
	Windows Server 2012 R2 | October 10 2023 
	Windows Server 2016 | January 12 202
	Windows Server 2019 | Janaru 9 2029```
Query Patch level:
	`wmic qfe`
	Sherlock script
		` Set-ExecutionPolicy bypass -Scope process`
		`Import-Module .\Sherlock.ps1`
		`Find-AllVulns`
[Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
	Feed systeminfo into textfile and add to script.
	`python2.7 windows-exploit-suggester.py  --database 2021-05-13-mssb.xls --systeminfo win7lpe-systeminfo.txt`	