**Lightweight Directory Access Protocol**
Default ports 389, 636, 3268, 3269 

* Stores information in hierarchical directories and allows users to lookup any information or devices in a network.
* Multiple servers across a network can have the same LDAP directory 
## Enumerate
```shell
nmap -n -sV --script "ldap* and not brute" $IP-DNS
```
