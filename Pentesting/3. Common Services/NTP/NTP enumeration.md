**Network Time protocol**
* Used for clock synchronisation 
* UDP port 123

### Enumeration 
Show peers system
```shell
ntpdc -c listpeers
```
Show NTP associations
```shell
ntpw -c associations [ip address]
```
Nmap
```shell
nmap -sU -sV --script "ntp* and (discovery or vuln) and not (dos or brute)" -p 123 [i]
```


