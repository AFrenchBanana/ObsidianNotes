# SSRF
* Works on API requests 
* Designed to exploit how a server behaves cause it to make requests to an external resource.
* If a target is accessing external resources via URL, and an attacker can modify that URL, they can generate a request to their own malicious infrastructure and capture information or trigger RCE.
* SSRF is not restricted to just external URLs. Can exploit access on localhost. 
* Web applications can receive input from an external URL. May be none/ minimal validation on the data.  May be able to trigger SQLi or buffer overflows etc..
### Use case 
#### SSRF Attacks
Exploiting a processes in which the client accesses a resource via a URL. The attacker will attempt to force the web application to make a HTTP request back to itself via its loopback interface. Potentially Revealing data

#### SSRF Back-End Attack
Occurs when a back-end component has full access rights due to a trusted relationship. An attacker could potentially forge a request and gain access to sensitive data.
These attacks can run port scans

#### Example
Intended request:
```http request 
POST /item/stpcl HTTP/1.0 Content-Type/x-www-form-urlencoded
stockCheckapi=http://example.com/item/stock/check/item123
```
Malicious request
```http request 
POST /item/stpcl HTTP/1.0 Content-Type/x-www-form-urlencoded
stockCheckapi=http://localhost/admin
```
This may bypass security measures as it comes from the local machine. 
Local host could also be another IP. 

## Filter Bypass techniques 
Alternate inputs to by pass filters:

### Alternate ways to do localhost
| Use Case                 | Method                   |
| ------------------------ | ------------------------ |
| All IPv4                 | 0                        |
| All IPv6                 | ::                       |
| All IPv4                 | 0.0.0.0                  |
| LocalHost IPv6           | ::1                      |
| All IPv4                 | 0000                     |
| All IPv4(Leading Zeros)  | 00000000                 |
| IPv4 Mapped IPv6 address | 0:0:0:0:0:FFFF:7F00:0001 |
| 8-bit octal conversion   | 0177.00.00.01            |
| 32 Bit Octal Conversion  | 017700000001             |
| 32 Bit Hex Conversion    | 0x7f000001               |


# CSRF
* Designed to force a legitimate, authenticated user to take malicious or undesirable actions on a webapp.
* Authentication is often stored in a client-side cookie. The cookie is then used to maintain authentication. 
* Attacker can force the clients browser to access the valid site and authenticate with its cookie. 
* Common affects include changing passwords and scraping sensitive information.
#### Use Case
good request
```http request 
GET http://bank.com/transfer?to=Alice&amount=50.00 HTTP/1.1
```
Manipulated request 
```http request 
GET http://bank.com/transfer?to=Alice&amount=100.00 HTTP/1.1
```
Need a valid authentication cookie for this to work

#### Example
With a Get request
```html
<img src="http://bank.com/transfer?to=Steve&amount=100" width="0" height="0" border="0">
```

```html
<form action="http://bank.com/transfer" method="POST">
	<input type="hidden" name="to" value="Steve"/>
	<input type="hidden" name="amount" value="100"/>
	<input type="submit" value="Click me!"/>
</form>
```

### Same-Origin Policy 
* Browser-level security control which dictates how a document or script served by one origin can interact with a resource from some other origin
* Prevents scripts running on one site to read data from another side
* Cross-domain requests and form submissions are still permitted but reading data is not.
* **Only Prevents you reading data**

# Key Differences
## Attack target
* SSRF the target is the server
* CSRF targets a user 
## Attack Intent
* SSRF primary intent is to access and ex-filtrate information.
* CSRF has no ex filtration of data.  Attacker does not receive any feedback or output

# Defences
## URL Schemas
* Specify the protocol which should be used when requesting a resource. 
* SSRF attackers can use different file schema's like `ftp://`, `smb://`
* URL schema can prevent this

