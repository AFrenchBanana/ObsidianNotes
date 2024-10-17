Open Source Ticket Tracker than can be compared to systems such as Jira, OTRS, Request Tracker. 
# Footprinting and Enumeration 
* Web application, Nmap will just return the web server such as Apache or IIS.
* Only staff and users with administrator privileges can access the admin panel. 
* Try and "play dumb" and contact the company with a social engineering attack. 
* Create a problem and report it and you are likely to get more emails to use against the admin panel. 
# Attacking osTicket
* Create requests and look for legitimate emails.
* Can you then use this email to register on another service such as Slack, Matter-most, Git, etc...
* Use this email to register an account and the help desk support portal to receive a signup confirmation email.
## Sensitive Data Exposure
Find credentials being used on Deshed for example:
```shell
sudo python3 dehashed.py -q inlanefreight.local -p
```
Another useful tool:
https://github.com/initstring/linkedin2username

* Use the emails/ usernames and passwords on the support system.
* Look within the the tickets for sensitive data such as credentials. 
