## Google Dorking
```Google Dork 
(intitle:<title> & version)filetype:<filetype>
```
```google
# search for devices connected to the internet such as public cameras
inurl:"ViewerFrame?Mode="
```

### Google hacking database
ExploitDB google hacking database (GHDB)

## WHOIS
Queries RFC 3912 database of an IP 
```shell
whois <domain>
```
## NSLOOKUP
Obtain a domain name from an IP address
## Shodan 
* Can search for online devices 
* search engine of service banners
* Can collect on ports:
	* 21
	* 22
	* 23
	* 25
	* 80/8080
	* 161
	* 443/8443


* Search Filters

| Filter Name      | Description                                           | Example                  |
| ---------------- | ----------------------------------------------------- | ------------------------ |
| city             | search devices in a specified city                    | `city:"London"`          |
| country          | Search in specified country                           | `country:GB`             |
| http.title       | This will search for specific website titles          | `http.title:"hacked by"` |
| net              | This will look for all devices in a network range     | `net:8.8.0.0/16`         |
| org              | This will search for devices owned by an organisation | `org.Google`             |
| port             | This will search all devices on a port                | `port:22`                |
| screenshot.label | Search for any devices that have screenshots attached | `screenshot.label`       |

