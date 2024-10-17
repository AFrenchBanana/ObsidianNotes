#SMB
## Interacting with on Windows 
### CMD
```cmd
dir \\192.168.220.129\Finance\
```
Connect/ Disconnect a computer
Map to n: drive
```cmd
net use n: \\192.168.220.129\Finance
```

```cmd
net use n: \\192.168.220.129\Finance /user:plaintext Password123
```
List files in that directory 
```cmd
dir n: /a-d /s /b | find /c ":\"
```

|**Syntax**|**Description**|
|---|---|
|`dir`|Application|
|`n:`|Directory or drive to search|
|`/a-d`|`/a` is the attribute and `-d` means not directories|
|`/s`|Displays files in a specified directory and all subdirectories|
|`/b`|Uses bare format (no heading information or summary)|

#### Find Useful Files 
Files with word cred in: 
```cmd
dir n:\*cred* /s /b
```

```cmd
findstr /s /i cred n:\*.*
```

### PowerShell

```powershell
Get-ChildItem \\192.168.220.129\Finance\
```
Mount Drive 
To Provide username and password need to create a [PSCredential](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.pscredential?view=powershellsdk-7.3.0) object: 
```powershell
$username = 'plaintext'

$password = 'Password123'

$secpassword = ConvertTo-SecureString $password -AsPlainText -Force

$cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
```

```powershell
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"
```

Count Files in drive:
*need to be in the right path*
```powershell
(Get-ChildItem -File -Recurse | Measure-Object).Count
```

Find files with the word cred in them
```powershell
Get-ChildItem -Recurse -Path N:\ -Include *cred* -File
```
Select string 
```powershell
Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List
```

## Interacting with Linux

**Need CIFS-UTIL installed**
`sudo apt install cifs-utils`.

```shell
sudo mkdir <file path>
```

```shell
sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance <directory>
```

Alternative with a cred file 
```shell
 mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

```txt
username=plaintext
password=Password123
domain=.
```

#### Find Files

```shell
find /mnt/Finance/ -name *cred*
```

```shell
grep -rn /mnt/Finance/ -ie cred
```
