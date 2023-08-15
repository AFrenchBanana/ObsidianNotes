### Simple Network Management Protocol 

Used to monitor network devices.
Transmits control commands on UDP 161
Transmits traps on UDP 162

## Management Information Base (MIB)

Allows #SNMP to work across manufactures. 
Text file in which all query-able SNMP objects are listed. 
Written in Abstract Syntax Notation One (ASN.1)
Do not contain data but where to find information and what it looks like.
#### OID 
Object Identifier
Node in a hierarchical namespace.
Sequence of numbers identifies each node, allowing the nodes positions to be determined. 


## Versions 
### SNMPv1
Still in use on many small networks. 
No built in Authentication 
	Anyone can read and modify network data
Does not support encryption 

### SNMPv2
Version today is **v2c**
	C stands for community based.
Same security as v1

### SNMPv3 
Authentication features using username and passwords 
transmission encryption via a pre-shared key.
More configuration options

# Configuration and Settings
## Config
```shell-session
cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'
```
## Dangerous Settings 

|**Settings**|**Description**|
|---|---|
|`rwuser noauth`|Provides access to the full OID tree without authentication.|
|`rwcommunity <community string> <IPv4 address>`|Provides access to the full OID tree regardless of where the requests were sent from.|
|`rwcommunity6 <community string> <IPv6 address>`|Same access as with `rwcommunity` with the difference of using IPv6.|

# Footprinting

### #snmpwalk
```shell-session
snmpwalk -v2c -c public <ip>
```

### OneSixtyOne
```shell-session
sudo apt install onesixtyone

onesixtyone -c /opt/useful/SecLists/Discovery/SNMP/snmp.txt 10.129.14.128
```

### Braa 
```shell-session
sudo apt install braa

braa <community string>@<IP>:.1.
3.6.*  #dont change <ip> here

braa public@<ip>:.1.3.6.*
```
