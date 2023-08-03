## Active Infrastructure Identification 

What operating system is being run?
- IIS 6.0: Windows Server 2003
- IIS 7.0-8.5: Windows Server 2008 / Windows Server 2008R2
- IIS 10.0 (v1607-v1709): Windows Server 2016
- IIS 10.0 (v1809-): Windows Server 2019

### Web Servers
```shell-session
curl -I "http://${TARGET}"
```

### Cookies 
- .NET: `ASPSESSIONID<RANDOM>=<COOKIE_VALUE>`
- PHP: `PHPSESSID=<COOKIE_VALUE>`
- JAVA: `JSESSION=<COOKIE_VALUE>`

#### WhatWeb
```shell-session
 whatweb -a3 https://www.facebook.com -v
```

### WafW00f
Web application firewall (WAF) fingerprinting tool
```shell-session
sudo apt install wafw00f -y
```

```shell-session
wafw00f -v https://www.tesla.com
```

### Aquatone
Automatic and visual inspections across sites.

```shell-session
sudo apt install golang chromium-driver

go get github.com/michenriksen/aquatone

export PATH="$PATH":"$HOME/go/bin"
```
```shell-session
aquatone --help
```

```shell-session
cat <subdomains> | aquatone -out ./aquatone -screenshot-timeout 1000
```
returns a aquatone_report.html file

## Active Subdomain Enumeration 

Identify Name Servers
```shell-session
nslookup -type=NS <ip>
```

Test for zone transfers
```shell-session
nslookup -type=any -query=AXFR zonetransfer.me nsztm1.digi.ninja
```

Gobuster 
```shell-session
Gobuster dns -q -r "${NS}" -d "${TARGET}" -w "${WORDLIST}" -p ./patterns.txt -o "gobuster_${TARGET}.txt"
```

## Virtual Hosts

Allows several hosts to be hosted on one server. 

#### IP-Based Virtual Hosting 
Multiple network interfaces with own IP addresses.

#### Name-based Virtual Hosting 
Made at the application level. 
admin.x.x 
backup.x.x

```shell-session
cat ./vhosts | while read vhost;do echo "\n********\nFUZZING: ${vhost}\n********";curl -s -I http://192.168.10.10 -H "HOST: ${vhost}.randomtarget.com" | grep "Content-Length: ";done
```

ffuf or gobuster?

```shell-session
ffuf -w Mwordlist> -u http://<ip> -H "HOST: FUZZ.randomtarget.com" -fs 612
```

## Crawling 

List all resources encountered. 
### ZAP spidering function 
![[ZAPSpidering.png]]

### FFuF
```shell-session
ffuf -recursion -recursion-depth 1 -u http://<ip>/FUZZ -w <wordlist>
```
```raft-[ small | medium | large ]-extensions.txt>)``` 
SECLISTS


```shell-session
 cewl -m5 --lowercase -w wordlist.txt http://<ip>
```

```shell-session
ffuf -w ./folders.txt:FOLDERS,./wordlist.txt:WORDLIST,./extensions.txt:EXTENSIONS -u http://<ip>/FOLDERS/WORDLISTEXTENSIONS
```

```shell-session
curl http://<ip><path>
```
