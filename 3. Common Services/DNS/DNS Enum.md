Port 53
# Types of #DNS

|**Server Type**|**Description**|
|---|---|
|`DNS Root Server`|The root servers of the DNS are responsible for the top-level domains (`TLD`). As the last instance, they are only requested if the name server does not respond. Thus, a root server is a central interface between users and content on the Internet, as it links domain and IP address. The [Internet Corporation for Assigned Names and Numbers](https://www.icann.org/) (`ICANN`) coordinates the work of the root name servers. There are `13` such root servers around the globe.|
|`Authoritative Nameserver`|Authoritative name servers hold authority for a particular zone. They only answer queries from their area of responsibility, and their information is binding. If an authoritative name server cannot answer a client's query, the root name server takes over at that point.|
|`Non-authoritative Nameserver`|Non-authoritative name servers are not responsible for a particular DNS zone. Instead, they collect information on specific DNS zones themselves, which is done using recursive or iterative DNS querying.|
|`Caching DNS Server`|Caching DNS servers cache information from other name servers for a specified period. The authoritative name server determines the duration of this storage.|
|`Forwarding Server`|Forwarding servers perform only one function: they forward DNS queries to another DNS server.|
|`Resolver`|Resolvers are not authoritative DNS servers but perform name resolution locally in the computer or router.|

Typically encrypted, unless using DNS over TLS(DoT) or DNS over #HTTPS (DoH)

![[DNS.png]]

# DNS Records 
#Records

|**DNS Record**|**Description**|
|---|---|
|`A`|Returns an IPv4 address of the requested domain as a result.|
|`AAAA`|Returns an IPv6 address of the requested domain.|
|`MX`|Returns the responsible mail servers as a result.|
|`NS`|Returns the DNS servers (nameservers) of the domain.|
|`TXT`|This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam.|
|`CNAME`|This record serves as an alias. If the domain www.hackthebox.eu should point to the same IP, and we create an A record for one and a CNAME record for the other.|
|`PTR`|The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.|
|`SOA`|Provides information about the corresponding DNS zone and email address of the administrative contact.|

[**Bind9**]([https://www.isc.org/bind/](https://wiki.debian.org/Bind9))

# Common DNS configuration
 `named.conf.local`
 `named.conf.options`
 `named.conf.log`

### Local DNS Configuration
```shell-session
cat /etc/bind/named.conf.local
```

### Zone Files
```shell-session
/etc/bind/db.domain.com
```
### Reverse Name Resolution
```shell-session
/etc/bind/<ip>
```

### Dangerous Settings

|**Option**|**Description**|
|---|---|
|`allow-query`|Defines which hosts are allowed to send requests to the DNS server.|
|`allow-recursion`|Defines which hosts are allowed to send recursive requests to the DNS server.|
|`allow-transfer`|Defines which hosts are allowed to receive zone transfers from the DNS server.|
|`zone-statistics`|Collects statistical data of zones.|
# DNS enumeration 
### Grab DNS details
```shell
dig version.bind CHAOS TXT @DNS
```
```shell
nmap --script dns-nsid [ip]
```
### Metasploit 
```msf
auxiliary/gather/enum_dns
```
### DIG
#DIG
Reverse Lookup
```shell
dig -x [ip] @[dnsip]
```
DNS that resolves that name
```shell
dig ns <url> @<ip>
```
Information 
```shell
dig TXT @[dnsip] [domain]
```
Any information
```
dig any [dnsip] <ip>
```

```
dig soa <domain>
```
#### Zone Transfer check 
```shell
dnsrecon -d [domain-name] -t axfr
```
With domain
```shell
dig axfr <domain> @<ip>
```
without domain 
```shell
dig axfr [dnsip]
```

```shell
fierce --domain [domain] --dns-servers [dnsip]
```
#### Zone Transfer Internal
```shell
dig axfr internal.<domain> @<ip>
```
#### Reverse IP lookup 
```shell
dnsrecon -d [domain name] -s
```
All addresses
```
dnsrecon -r [ip-dns/24] -n [dns-ip]
```
## Subdomain Brute Forcing
#### Locate subdomains 
```shell
dnsrecon -D subdomains-1000.txt -d $domain-name -n [ip]
```
```shell
nmap --script dns-srv-enum --script-args "dns-srv-enum.domain='domain.com'"
```

```shell-session
dnsenum --dnsserver <ip> --enum -p 0 -s 0 -o subdomains.txt -f <wordlist> <domain>
```
Can you #bruteforce  #subdomains from the zone transfer?