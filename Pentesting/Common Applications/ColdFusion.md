## What is it
* Programming language and web application development based on Java.
* Used to build dynamic and interactive web applications that can be connected to various APIs. 
* ColdFusion Markup Language (CFML) is the language used by ColdFusion to develop dynamic web pages
	* This works similar to HTML.
	* Works on tags and has more functionality such as execute SQL commands
* Retrieve records in databases
```html
<cfquery name="myQuery" datasource="myDataSource">
  SELECT *
  FROM myTable
</cfquery>
```
* Iterate through records:
```html
<cfloop query="myQuery">
  <p>#myQuery.firstName# #myQuery.lastName#</p>
</cfloop>
```
* This allows Complex logic with minimal code

|**Benefits**|**Description**|
|---|---|
|`Developing data-driven web applications`|ColdFusion allows developers to build rich, responsive web applications easily. It offers session management, form handling, debugging, and more features. ColdFusion allows you to leverage your existing knowledge of the language and combines it with advanced features to help you build robust web applications quickly.|
|`Integrating with databases`|ColdFusion easily integrates with databases such as Oracle, SQL Server, and MySQL. ColdFusion provides advanced database connectivity and is designed to make it easy to retrieve, manipulate, and view data from a database and the web.|
|`Simplifying web content management`|One of the primary goals of ColdFusion is to streamline web content management. The platform offers dynamic HTML generation and simplifies form creation, URL rewriting, file uploading, and handling of large forms. Furthermore, ColdFusion also supports AJAX by automatically handling the serialisation and deserialisation of AJAX-enabled components.|
|`Performance`|ColdFusion is designed to be highly performant and is optimised for low latency and high throughput. It can handle a large number of simultaneous requests while maintaining a high level of performance.|
|`Collaboration`|ColdFusion offers features that allow developers to work together on projects in real-time. This includes code sharing, debugging, version control, and more. This allows for faster and more efficient development, reduced time-to-market and quicker delivery of projects.|

## Discovery & Enumeration
### Common Open Ports
|Port Number|Protocol|Description|
|---|---|---|
|80|HTTP|Used for non-secure HTTP communication between the web server and web browser.|
|443|HTTPS|Used for secure HTTP communication between the web server and web browser. Encrypts the communication between the web server and web browser.|
|1935|RPC|Used for client-server communication. Remote Procedure Call (RPC) protocol allows a program to request information from another program on a different network device.|
|25|SMTP|Simple Mail Transfer Protocol (SMTP) is used for sending email messages.|
|8500|SSL|Used for server communication via Secure Socket Layer (SSL).|
|5500|Server Monitor|Used for remote administration of the ColdFusion server.|
### Enumeration 

| **Method**        | **Description**                                                                                                                                                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Port Scanning`   | ColdFusion typically uses port 80 for HTTP and port 443 for HTTPS by default. So, scanning for these ports may indicate the presence of a ColdFusion server. Nmap might be able to identify ColdFusion during a services scan specifically. |
| `File Extensions` | ColdFusion pages typically use ".cfm" or ".cfc" file extensions. If you find pages with these file extensions, it could be an indicator that the application is using ColdFusion.                                                           |
| `HTTP Headers`    | Check the HTTP response headers of the web application. ColdFusion typically sets specific headers, such as "Server: ColdFusion" or "X-Powered-By: ColdFusion", that can help identify the technology being used.                           |
| `Error Messages`  | If the application uses ColdFusion and there are errors, the error messages may contain references to ColdFusion-specific tags or functions.                                                                                                |
| `Default Files`   | ColdFusion creates several default files during installation, such as "admin.cfm" or "CFIDE/administrator/index.cfm". Finding these files on the web server may indicate that the web application runs on ColdFusion.                       |

* Navigating to the various open ports may show off files such as `CFIDE` and `CFDOCS`
* Look for `.cfm` files 
## Attacking ColdFusion
* Searchsploit has a number of exploits
```shell
searchsploit adobe coldfusion
```
Passwords are stored in:
`lib/password.properties`
* Path traversal