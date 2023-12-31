Port 139,445

#### nmap commands 
```shell
sudo nmap <ip> -sV -sC -p139,445
```
Determines OS, computer name, domain and workgroup
```shell
nmap --script smb-os-discovery --open -p 139 [ip]
```
Output to XML
```shell
nmap --script smb-os-discovery --open -p 139 [ip] -oX smb.xml
```
Enumerate Shares
```shell
nmap --script smb-enum-shares -p 139 [ip]
```
Check SMB vulnerabilities
```shell
nmap -p 139,445 -vv -Pn --script=smb-vuln* [IP]
```

Access rights designed as Access Control Lists (ACL)
#Samba
	Alternate to #SMB 
	Ports 137,138,139 or 445
	
|**SMB Version**|**Supported**|**Features**|
|---|---|---|
|CIFS|Windows NT 4.0|Communication via NetBIOS interface|
|SMB 1.0|Windows 2000|Direct connection via TCP|
|SMB 2.0|Windows Vista, Windows Server 2008|Performance upgrades, improved message signing, caching feature|
|SMB 2.1|Windows 7, Windows Server 2008 R2|Locking mechanisms|
|SMB 3.0|Windows 8, Windows Server 2012|Multichannel connections, end-to-end encryption, remote storage access|
|SMB 3.0.2|Windows 8.1, Windows Server 2012 R2||
|SMB 3.1.1|Windows 10, Windows Server 2016|Integrity checking, AES-128 encryption|

Default configuration
```shell
cat /etc/samba/smb.conf | grep -v "#\|\;" 
```

**Setting Options**

|**Setting**|**Description**|
|---|---|
|`[sharename]`|The name of the network share.|
|`workgroup = WORKGROUP/DOMAIN`|Workgroup that will appear when clients query.|
|`path = /path/here/`|The directory to which user is to be given access.|
|`server string = STRING`|The string that will show up when a connection is initiated.|
|`unix password sync = yes`|Synchronize the UNIX password with the SMB password?|
|`usershare allow guests = yes`|Allow non-authenticated users to access defined shared?|
|`map to guest = bad user`|What to do when a user login request doesn't match a valid UNIX user?|
|`browseable = yes`|Should this share be shown in the list of available shares?|
|`guest ok = yes`|Allow connecting to the service without using a password?|
|`read only = yes`|Allow users to read files only?|
|`create mask = 0700`|What permissions need to be set for newly created files?|

***Dangerous Settings***

|**Setting**|**Description**|
|---|---|
|`browseable = yes`|Allow listing available shares in the current share?|
|`read only = no`|Forbid the creation and modification of files?|
|`writable = yes`|Allow users to create and modify files?|
|`guest ok = yes`|Allow connecting to the service without using a password?|
|`enable privileges = yes`|Honor privileges assigned to specific SID?|
|`create mask = 0777`|What permissions must be assigned to the newly created files?|
|`directory mask = 0777`|What permissions must be assigned to the newly created directories?|
|`logon script = script.sh`|What script needs to be executed on the user's login?|
|`magic script = script.sh`|Which script should be executed when the script gets closed?|
|`magic output = script.out`|Where the output of the magic script needs to be stored?|
### Commands
#### Restart Samba
```shell
sudo systemctl restart smbd
```
#### Samba Status 
```shell
smbstatus
```


#### Connect to a #Share 
list shares
```shell
 smbclient -N -L //<ip>
```
Connect to share
```shell
smbclient //<ip>/share
```
Recursive get 
```shell
smbget -a smb://[ip]/[file] -R
```
Upload a file 
```shell
smbmap -H [ip] --upload [file] [share]
```
List shares as Guest 
```shell
smbmap -H [ip] -u Guest -R
```
List Shares 

#### RPCClient 
```shell
rpcclient -U "" <ip>
```

|**Query**|**Description**|
|---|---|
|`srvinfo`|Server information.|
|`enumdomains`|Enumerate all domains that are deployed in the network.|
|`querydominfo`|Provides domain, server, and user information of deployed domains.|
|`netshareenumall`|Enumerates all available shares.|
|`netsharegetinfo <share>`|Provides information about a specific share.|
|`enumdomusers`|Enumerates all domain users.|
|`queryuser <RID>`|Provides information about a specific user.|

Brute Force RID
```shell
for i in $(seq 500 1100);do rpcclient -N -U "" <ip> -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```
```shell
impacket-samrdump <ip>
```

#SMBmap
```shell
smbmap -H <ip>
```

#CrackMapExec
```shell
crackmapexec smb <ip> --shares -u '' -p ''
```

[Enum4Linux-ng](https://github.com/cddmp/enum4linux-ng)
```shell
./enum4linux-ng.py <ip> -A
```
