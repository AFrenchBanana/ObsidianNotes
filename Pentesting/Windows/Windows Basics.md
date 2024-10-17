# Files and Folders
## Folders of Interest 
| Folder                                                  | Notes                                                                      | Linux Equivalent                   |
| ------------------------------------------------------- | -------------------------------------------------------------------------- | ---------------------------------- |
| `C:\Program Files`                                      | Stores Programs and Applications                                           | `/bin` `/usr/bin`                  |
| `C:\Program Files (x86)`                                | Stores x86 and x64 versions of Windows                                     | `/bin` `/usr/bin`                  |
| `C:\Program Data`                                       | Contains Program data that is expected to be accessed by computer programs | `/home/user/.<app>`                |
| `C:\Users`                                              | Stores User account folders                                                | `/home`                            |
| `C:\Windows`                                            | Where windows is intalled                                                  | `/`                                |
| `C:\Windows\System` `C:\System32` `C:\Windows\SysWOW64` | Store DLL files that implement core features of Windows and Windows APIs.  | `/usr/lib` `/ussr/bin` `/usr/sbin` |

## Interacting with files 
| Command                        | Explanation                                             | Linux Equivalent |
| ------------------------------ | ------------------------------------------------------- | ---------------- |
| `mkdir`                        | Creates a directory                                     | `mkdir`          |
| `rmdir`                        | Removes a directory                                     | `rmdir`          |
| `ren`                          | Rename a directory or file                              | `mv`             |
| `type`                         | Read contents of a file                                 | `file`           |
| `echo`                         | Prints text to standard out                             | `echo`           |
| `copy`                         | Copies a folder or file                                 | `cp`             |
| `move`                         | Moves a folder or file                                  | `mv`             |
| `/?`                           | provides help                                           | `man`            |
| `more`                         | Same as type but displays info in pages with q to quit  |                  |
| `Get-Content` *(powershell)* | PowerShell command can add -head or -tail with a number |                  |
|`Out-Host -Paging` *ignore escape character* *(powershell)*|Read files in a similar format to less in linux|`Less`|

## File and Folder Permissions
| Acronym | Access           |
| ------- | ---------------- |
| `D`     | Delete           |
| `F`     | Full             |
| `N`     | No access        |
| `M`     | Modify           |
| `RX`    | Read and Execute |
| `R`     | Read only        |
| `W`     | Write Only       |

Can check permissions with icalcs.exe

# User groups and permissions 
### Local and Domain:
Categorised into belonging to:
1. 'Local' system
2. Work group
3. Domain
#### Local
Local user authentication occurs with the computer and without external verification

When you login as a local user, the computer queries to check if you are allowed to login before, then applying all permissions and restrictions that are applied to your account
#### Domain
Domain User authentication verifies user access using Active Directory Domain Services. 
Domain controller gets queried for privileges assigned to you
When the DC replies, the user is logged in with the relevant permissions and restrictions. 

### Users and Groups 
**Permissions are based on account and/ or group to files and resources**
#### Users
1. User accounts 
2. Administrator accounts 
3. Guest Accounts 
#### Groups
1. User groups 
2. Administrator groups

#### Local Users and Groups 
* Can be managed via the 'Computer management' application
* Local accounts exist on a single machine and cannot be used to access other devices on a network.
* Local users must conform to local security policies 
* Local administrators belong to Administrators group
* Local Users ave no access to domain resources. 

#### Domain Users and Groups 
* Configured within active directory
* Domain accounts can login to multiple servers/clients on the domain
* Domain users must conform to domain security policies 
* Domain admins group contains accounts permitted to administrator the domain. (Enterprise Domains in a forest.)

## Windows System 
* `SYSTEM` is the highest privilege level in the Windows user model.
* Provides ownership to objects created before a regular user logs on such as LSASS and Session Management Subsystem (SMSS)
* Is not a real account. But instead a security principle
* Pseudo-account, equivalent to root on Linux bar 2 differences:
	1. SYSTEM is a service account, and does not have a user profile
	2. Windows permissions model still enforces ACLs such as objects can exists which SYSTEM cannot directly access

## Command Linue
### Piping and redirecting output 
| Integer Value | File Stream | Name            | Redirect Method          |
| ------------- | ----------- | --------------- | ------------------------ |
| 0             | STDIN       | Standard Input  |                          |
| 1             | STDOUT      | Standard Output | new file `>` append `>>` |
| 2             | STDERROR    | Standard Error  | `2>`                     |

`echo "Error Message" 2> stderror.txt`

### Searching and Filtering in the Terminal 
#### CMD
`For` - equivalent to cut in Linux
- Example
	- `for /f "tokens=3, * delims=/" %a in (input.txt) do @echo %b > output.txt` 
	- Removes the first 3 characters of a list separated by a / then outputs to a file.
`sort` - sort rows of output into an order of choice 
* Example
	* `sort /R input.txt /output.txt`
	* Reverses the the file.
#### PowerShell
`split` - cmd equivalent of `tr`
* Example
	* `$a= ( echo $env:Path | Out-String`
	* `$a -split ";"`
	* removes the character ;
## Registry 
* Database that stores settings for Windows and installed applications that choose to use it. 
* Changes within Windows mostly are always stored in the registry, changing manually often needs a reboot.
* By default admins have fill access to the registry whereas standard users have read access except for HKCU which is write. 
### Registry Design 
#### Hives 
* Microsoft describes a hive in the registry as the name given to a major section of the registry that contains keys, sub-keys and values. 
* All keys that are considered hives started with HKEY and are at root level. 
	* Sometimes called root keys or core system hives.
#### Registry Keys 
* Can be thought of being a bit like a file folder but exist only in the windows registry, Contain registry values, just like folders contain files. 
* Keys can also contain other keys called sub-keys
#### Registry Values
* Full of objects called values that contain specific instructions that windows applications refer to
* Many kinds of registry values exist, such as `DWORD (32-bit)` or `QWORD (64-bit)`
### Key Registry Locations
| Location                                                               | Description      |
| ---------------------------------------------------------------------- | ---------------- |
| `HKLM\Software\Microsoft\Windows\NT\CurrentVersion`                    | OS Info          |
| `HKLM\Software\Microsoft\Windows\NT\CurrentVersion /v RegisteredOwner` | Registered Owner |
| `HKLM\Software\Microsoft\Windows\NT\CurrentVersion /v SystemRoot`      | System Root      |
| `HKLM\System\MountedDevices`                                           | Mounted Devices  |
| `HKLM\Security\Policy\PolAdTev`                                        | Audit Policies   |
| `HKLM\Software`                                                        | Default Software |
| `HKCU\Software`|User Installed Software|

### Useful Commands and Files
#### Scheduled Tasks (CMD)
| Command           | Description                  |
| ----------------- | ---------------------------- |
| `schtasks query`  | Displays Scheduled tasks     |
| `schtaks create`  | Creates a new scheduled task |
| `schtasks end`    | End a task                   |
| `schtasks delete` | Deletes a task               |
#### Network Commands 
| Command              | Example Usage         |
| -------------------- | --------------------- |
| `ssh [user]@[ip]`    | ssh user@192.168.1.1  |
| `telnet [ip] [port]` | telnet 192.168.1.1 80 |
| `ping [ip]`          | ping 192.168.1.1      |
| `ipconfig /all`      |                       |
| `netstat -a`|                       |
#### User Information 
| command                                                          | description                              |
| ---------------------------------------------------------------- | ---------------------------------------- |
| `net user`                                                       | Lists user accounts on the machine       |
| `net user /add`                                                  | Adds a user                              |
| `net user /delete`                                               | Deletes a User                           |
| `net localgroup`                                                 | Shows groups on system                   |
| `Get-LocalGroup \| ft Name`                                       | (powershell) *ignore escape char*                             |
| `net localgroup Administrators`                                  | Shows users in administrator group       |
| `Get-LocalGroupMember Administrators \| ft Name, PrinicpalSource` | (powershell) *ignore escape char*                             |
| `whoami`                                                         | shows the current username               |
| `whoami /priv`                                                   | Shows the current users privilges        |
| `whoami /groups`                                                 | Shows the current users group membership |
#### System Information (CMD)
| system information            | Description            |
| ----------------------------- | ---------------------- |
| `systeminfo`                  | Lists system info      |
| `tasklist`                    | list running processes |
| `taskkill /f /im process.exe` | kill a process         |
#### Service Management (CMD)
| Command                          | Command Description    |
| -------------------------------- | ---------------------- |
| `sc query state = all`           | View installed service |
| `net start 'service'`            | starts a service       |
| `sc start 'service'`             | Starts a service       |
| `net stop 'service'`             | Stops a service        |
| `sc stop service`                | Stops a service        |
| `sc config 'service' start=auto` | Enables a service      |
| `sc config 'service'`            | Disables a service     |
#### Important Files 
| Location                                   | File                    |
| ------------------------------------------ | ----------------------- |
| `C:\Windows\System32\drivers\etc\hosts`    | DNS File                |
| `C:\Windows\System32\drivers\etc\networks` | Networks config file    |
| `C:\Windows\System32\Config\SAM`           | usernames and passwords |
| `C:\Windows\repair\SAM`                    | SAM Backup              |

