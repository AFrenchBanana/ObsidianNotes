## WHOIS

```shell-session
export TARGET="facebook.com" # Assign our target to an environment variable
whois $TARGET
```

```cmd-session
whois.exe facebook.com
```

## DNS 

|   |   |
|---|---|
|`Resource Record`|A domain name, usually a fully qualified domain name, is the first part of a Resource Record. If you don't use a fully qualified domain name, the zone's name where the record is located will be appended to the end of the name.|
|`TTL`|In seconds, the Time-To-Live (`TTL`) defaults to the minimum value specified in the SOA record.|
|`Record Class`|Internet, Hesiod, or Chaos|
|`Start Of Authority` (`SOA`)|It should be first in a zone file because it indicates the start of a zone. Each zone can only have one `SOA` record, and additionally, it contains the zone's values, such as a serial number and multiple expiration timeouts.|
|`Name Servers` (`NS`)|The distributed database is bound together by `NS` Records. They are in charge of a zone's authoritative name server and the authority for a child zone to a name server.|
|`IPv4 Addresses` (`A`)|The A record is only a mapping between a hostname and an IP address. 'Forward' zones are those with `A` records.|
|`Pointer` (`PTR`)|The PTR record is a mapping between an IP address and a hostname. 'Reverse' zones are those that have `PTR` records.|
|`Canonical Name` (`CNAME`)|An alias hostname is mapped to an `A` record hostname using the `CNAME` record.|
|`Mail Exchange` (`MX`)|The `MX` record identifies a host that will accept emails for a specific host. A priority value has been assigned to the specified host. Multiple MX records can exist on the same host, and a prioritized list is made consisting of the records for a specific host.|


```shell-session
nslookup $TARGET
```

```shell-session
dig facebook.com @1.1.1.1
```

### Query A Records for subdomains 
```shell-session
export TARGET=www.facebook.com

nslookup -query=A $TARGET
```

```shell-session
dig a www.facebook.com @1.1.1.1
```

### Query PTR Records 
```shell-session
nslookup -query=PTR 31.13.92.36
```

```shell-session
dig -x 31.13.92.36 @1.1.1.1
```

### Query any record 
```shell-session
export TARGET="google.com"

nslookup -query=ANY $TARGET
```

```shell-session
dig any google.com @8.8.8.8
```

ANY DNS records are being abolished as per [RFC8482](https://datatracker.ietf.org/doc/html/rfc8482). So may not get results. 

### TXT Records 
```shell-session
nslookup -query=TXT $TARGET
```
```shell-session
dig txt facebook.com @1.1.1.1
```

### mx Records

```shell-session
dig mx facebook.com @1.1.1.1
```

## Passive Subdomain Enumeration 

[Virus Total](https://www.virustotal.com/gui/home/upload)

### Certificates 
https://censys.com/
[https://crt.sh](https://crt.sh)

#### Download Certificates in JSON

```shell-session
export TARGET="facebook.com"

curl -s "https://crt.sh/?q=${TARGET}&output=json" | jq -r '.[] | "\(.name_value)\n\(.common_name)"' | sort -u > "${TARGET}_crt.sh.txt"
```

|   |   |
|---|---|
|`curl -s`|Issue the request with minimal output.|
|`https://crt.sh/?q=<DOMAIN>&output=json`|Ask for the json output.|
|`jq -r '.[]' "\(.name_value)\n\(.common_name)"'`|Process the json output and print certificate's name value and common name one per line.|
|`sort -u`|Sort alphabetically the output provided and removes duplicates.|

#### Download using OpenSSL

```shell-session
export TARGET="facebook.com"

export PORT="443"

openssl s_client -ign_eof 2>/dev/null <<<$'HEAD / HTTP/1.0\r\n\r' -connect "${TARGET}:${PORT}" | openssl x509 -noout -text -in - | grep 'DNS' | sed -e 's|DNS:|\n|g' -e 's|^\*.*||g' | tr -d ',' | sort -u
```

### [The Harvester](https://github.com/laramies/theHarvester)

|   |   |
|---|---|
|[Baidu](http://www.baidu.com/)|Baidu search engine.|
|`Bufferoverun`|Uses data from Rapid7's Project Sonar - [www.rapid7.com/research/project-sonar/](http://www.rapid7.com/research/project-sonar/)|
|[Crtsh](https://crt.sh/)|Comodo Certificate search.|
|[Hackertarget](https://hackertarget.com/)|Online vulnerability scanners and network intelligence to help organizations.|
|`Otx`|AlienVault Open Threat Exchange - [https://otx.alienvault.com](https://otx.alienvault.com/)|
|[Rapiddns](https://rapiddns.io/)|DNS query tool, which makes querying subdomains or sites using the same IP easy.|
|[Sublist3r](https://github.com/aboul3la/Sublist3r)|Fast subdomains enumeration tool for penetration testers|
|[Threatcrowd](http://www.threatcrowd.org/)|Open source threat intelligence.|
|[Threatminer](https://www.threatminer.org/)|Data mining for threat intelligence.|
|`Trello`|Search Trello boards (Uses Google search)|
|[Urlscan](https://urlscan.io/)|A sandbox for the web that is a URL and website scanner.|
|`Vhost`|Bing virtual hosts search.|
|[Virustotal](https://www.virustotal.com/gui/home/search)|Domain search.|
|[Zoomeye](https://www.zoomeye.org/)|A Chinese version of Shodan.|

```shell-session
cat sources.txt

baidu
bufferoverun
crtsh
hackertarget
otx
projecdiscovery
rapiddns
sublist3r
threatcrowd
trello
urlscan
vhost
virustotal
zoomeye
```

```shell-session
export TARGET="facebook.com"

cat sources.txt | while read source; do theHarvester -d "${TARGET}" -b $source -f "${source}_${TARGET}";done
```

Extract Subdomains 

```shell-session
cat *.json | jq -r '.hosts[]' 2>/dev/null | cut -d':' -f 1 | sort -u > "${TARGET}_theHarvester.txt"
```

Merge all reconnaissance files 
```shell-session
 cat facebook.com_*.txt | sort -u > facebook.com_subdomains_passive.txt

cat facebook.com_subdomains_passive.txt | wc -l
```

## Passive Infrastructure Identification 

### Netcraft
Get information on servers without interacting with them.

[https://www.netcraft.com/](https://sitereport.netcraft.com)

|   |   |
|---|---|
|`Background`|General information about the domain, including the date it was first seen by Netcraft crawlers.|
|`Network`|Information about the netblock owner, hosting company, nameservers, etc.|
|`Hosting history`|Latest IPs used, webserver, and target OS.|

### Wayback Machine 
http://web.archive.org/

Download and inspect urls using wayback curls 
```shell-session
go install github.com/tomnomnom/waybackurls@latest
```

```shell-session
waybackurls -dates https://facebook.com > waybackurls.txt

cat waybackurls.txt
```
