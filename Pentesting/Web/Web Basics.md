# HTTP Requests 
## HTTP Components 
1. request line 
2. series of HTTP headers
3. message body

## Request Line
First line in HTTP request 
Contains:
1. A method i.e GET or POST 
2. URL path 
3. HTTP Version 
`POST /rest/user/login HTTP/1.1`

Can also contain additional lines:
* Full URL with protocol and host included `http://test.com/path`
* Post parameters `?key=value`

## Request Headers 
* Used to convey information from the client browser to the server.
* Headers sent as key-value pairs
	* key comes first, followed by a colon 
	* `"email":"User1"`
* Custom headers can be written

## Request Body
* Not always used but can be needed in some circumstances.
* Content length, cookies, user agent etc

## HTTP Status Codes
| Code    | Meaning                 |
| ------- | ----------------------- |
| 100-199 | Informational Responses |
| 200-299 | Success Responses       |
| 300-399 | Redirection Messages    |
| 400-499 | Client Error response   |
| 500-599 | Server Error response   |

# API Systems 
**Application Programming Interface**
* Usually work on a client/ server model
* Ways APIs can work
	1. SOAP APIs - Simple Object Access Protocol, messages exchanged with XML
	2. RPC APIs - Remote Procedure Calls, client completes a function on the server and the server sends the response
	3. Websocket APIs - Uses JSON objects to pass data, supports 2 way communications
	4. REST APIs - Client sends request to the server to run a function with required data, sever executes the command returns the data. (Most popular)
## API Security Issues
1. Multiple entry points - More ports open means more entry points. 
2. Inconsistent Request Formats - data needed changes over time so sanitation is often forgotten. 
3. Different methods of Access - API calls might come from scripted systems, IPT devices. This eliminates user-agent verification. 
4. Bad traffic can look good - Often can receive data that is technically valid but creates bad behaviour.

## HTML 

# HTML
**Hyper Text Markup Language**
### Structure 
`<!DOCTYPE html>` declaration as HTML 5 doc
`<html>` root element
`<head>` metadata of an element of a HTML page
`<title>` title for web page
`<body>` documents body 
`<h1>` defines a large heading
	`<h1>` to `<h6>` are smaller headings
`<p>` defines a paragraph 
`<a>` anchor tag to represent a hyperlink
`<img>` represent an image within HTML document 
# CSS
**Cascading Style Sheet**
Configure style, layout, colurs and other effects to HTML
`<link rel="stylesheet" href"<sheetname>.css">`
Format:
```css
body { 
	background-color: #f0f0f2;
	margin: 0;
	padding: 0;
	font-family: Helvertica
}
```
# JavaScript 
**Client-side scripting language**
* Can be written in with the html tags `<script>`
* Every line needs a ; 
* Uses keywords like `var` `let` `const`
```javascript
<script>
var a, b, c 
a = 4;
b = 5;
c = a+b;

document.getElementbyID("demo").innerHTML = 
"C = " + c + ".";
</script>
```
## JavaScript with HTML
```html 
<html>
<head>

<script src="MyScript.js"> </script>

<script src="/var/www/html/src/myScript.js"></script>

<script src="http://utl/myscript.js"></script>

</head>
</html>
```
## DOM
**Document Object Model**
* Programming interface for web documents
* Allows the page to change the document, style and content
* Data is represented as Nodes and Objects

### Interacting with the DOM 
```JavaScript
document.getElementbyID("demo").innerHTML = x
```
Allows text to be changed based or CSS properties to be changed etc.

# PHP
* Back-end technology
* Alternatives:
	Python
	Ruby
	Java
	.NET
	Goland
```PHP
<?php
	echo "Hello World";
?>
```
* Server side processing 
* Can receive data from forms
* generate dynamic page content
* work with databases 
* create sessions
* send and receive cookies
* Has many hash functions available to encrypt user data
	* makes it a secure and reliable language server
* Allows you to interact with database systems 
	* Mostly used in conjunction with PHP
### Basic Syntax 
Scripts being with `<?php` and end with `?>`

| Syntax    | example                                                  |
| --------- | -------------------------------------------------------- |
| Comments  | single line: `//` or `#` MultiLine: start: `*/` end `/*` |
| Variables | Each variable defined with `$`                           |
| echo      | `echo`                                                   |
| If/ Else  | `if ($x ==1{<do>;}`                                      |
| Loops     | `for ($x = 0; $x <= 10; $x++{echo "X = $x <br>";}`       |

### Interacting with databases 
Steps: 
1. Create a connection
2. Select database
3. Perform database query 
4. Use return data
5. close connection
```PHP
$result = my_sql_query("SELECT * FROM users", $connection);
if (!result){
die("Database failed")
}
```

# Protecting Server-Side Infrastructure
## Web Application Firewalls (WAF)
* Used to filter and monitor HTTP traffic between web applications and the internet in order to protect web applications 
* Typically prevents cross-site requrest forgery, XSS and L/RFI and SQLi
* Protocol Layer 7 defence 
## Data Sanitisation
```PHP
$str = "<script>alert('Cross Site Scripting')</script>";

$new str = filter_var($str
	FILTER_SANITIZE_STRING);

echo $newstr
```
