# Persistence
## Account Creation
```cmd
net user /add
```
* Attacker can use this to RDP, WinRM or SSH. 
## Scheduled Tasks
* Works the same way as cron jobs. 
View scheduled tasks
```cmd
schtasks /query
```
Create a task
```cmd
schtasks /CREATE /SC DAILY /TN "<NAME>" /TR "<what file to run>"
```
## Services
* Have a service that occurs whenever a machine beacons back or listens for a connection. 
* Requires at least admin rights, leaves lots of artifacts. 
```cmd
sc create <name> binpth= "cmd.exe /k <file>" start="auto" obj="LocalSystem"
```
```cmd
sc start persistence
```

```powershell
New-Service -Name "<name>" -BinaryPathName "<file to run>" -Description "<desc>" StartupType Automatic 
```
```powershell
sc start persistence 
```
# Logs
* Windows are in 5 categories 
	1. Application
	2. Security
	3. Setup
	4. System
	5. Forwarded Events
## Crash Dumps
* Located in `%SystemRoot%Minidump.dmp`
## Clearing up 
Change information the logs
```powershell
$(Get-Item <filename>).creationtime=$(Get-Date "mm/dd/yyyy")
```
```powershell
$(Get-Item <filename>).lastacccesstime=$(Get-Date "mm/dd/yyyy")
```
```powershell
$(Get-Item <filename>).lastwritetime=$(Get-Date "mm/dd/yyyy")
```

### Clear Logs with Metasploit
```msf
run clearlogs
```
Clear a specific type of log
```ruby
irb
log = client.sys.evenlog.open('system'
log.clear)
```

```
### Initial Actions
Disable powershell history
```Powershell
(Get-PSReadlineOption).HistorySavePath
```
#### Transfer files to 
```
%systemdrive%/
```
