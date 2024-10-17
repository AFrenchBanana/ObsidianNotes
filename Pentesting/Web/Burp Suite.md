## Configure Burp Suite
Browser extension called `FoxyProxy`
Allows proxy to be turned on or off 

Manual settings:
```Burp Proxy Settings
ProxyType: HTTP
IP Address: 127.0.0.1
Port: 8080
No Creds required
```
Need to manually install certificate authority, this can be found at `http://localhost:8080` and clicking on CA Certificate 

Install in browser: `Privacy & Security > Certificates > View Certificates`

## Target Panel
Where you can view a breakdown of the target
### Site map 
Any traffic in burp proxy will be recorded here 
### Scope
Allows you to define which hosts to ignore / collect

## Interceptor
Allows proxy to be enabled / disabled.

## Repeater
Allows an attacker to repeat a previous request.
Values can be changed manually. 

## Intruder
Allows an attacker to specify positions within requests where payloads can be inserted and sent to targets.
Can perform brute forcing attacks.

