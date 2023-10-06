**HTTP works on receiving verbs at the begging of the request**

| Verb      | Description                                                                                         |
| --------- | --------------------------------------------------------------------------------------------------- |
| `HEAD`    | Identical to a GET request, but its response only contains the `headers`, without the response body |
| `PUT`     | Writes the request payload to the specified location                                                |
| `DELETE`  | Deletes the resource at the specified location                                                      |
| `OPTIONS` | Shows different options accepted by a web server, like accepted HTTP verbs                          |
| `PATCH`   | Apply partial modifications to the resource at the specified location                               |

[All 9](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

## Insecure configurations 
```xml
<Limit GET POST>
    Require valid-user
</Limit>
```
Limits get and request to valid users, an attacker may still be able to use `HEAD`

## Insecure Coding
```php
$pattern = "/^[A-Za-z\s]+$/";

if(preg_match($pattern, $_GET["code"])) {
    $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'";
    ...SNIP...
}
```

* This sensitisation only works on `GET` parameter
* When the query is executed `$_REQUEST["code]` parameters are being used which may also contain `POST` parameters
* This leads to an inconsistency of HTTP verbs.
	* The `POST` request may be able to perform SQL injection

## Bypassing Basic Authentication 
### Identify 
1. Intercept a HTTP login request on BurpSuite
2. Right click on the request and click `change request method`
#### Identify What Requests the Server accepts:
```shell
 curl -i -X OPTIONS http://SERVER_IP:PORT/
```

```shell
HTTP/1.1 200 OK
Date: 
Server: Apache/2.4.41 (Ubuntu)
Allow: POST,OPTIONS,HEAD,GET
Content-Length: 0
Content-Type: httpd/unix-directory
```

Try steps 1 and 2 with these other verbs such as `HEAD`

## Bypassing Security Filters
* If only the `POST` paramter is being checked then you may be able to change to a `GET` request or a `GET` to a `POST`
