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
