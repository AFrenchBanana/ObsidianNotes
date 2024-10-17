**Lightweight Directory Access Protocol**
### Basics
* Is a protocol that contains information about network resources such as users groups computers printers. 
#### Pros

|**Functionality**|**Description**|
|---|---|
|`Efficient`|Efficient and fast queries and connections to directory services, thanks to its lean query language and non-normalised data storage.|
|`Global naming model`|Supports multiple independent directories with a global naming model that ensures unique entries.|
|`Extensible and flexible`|This helps to meet future and local requirements by allowing custom attributes and schemas.|
|`Compatibility`|It is compatible with many software products and platforms as it runs over TCP/IP and SSL directly, and it is `platform-independent`, suitable for use in heterogeneous environments with various operating systems.|
|`Authentication`|It provides `authentication` mechanisms that enable users to `sign on once` and access multiple resources on the server securely.|
#### Issues:

| Functionality | Description                                                                                                                                                                                                                       |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Compliance`  | Directory servers `must be LDAP compliant` for service to be deployed, which may `limit the choice` of vendors and products.                                                                                                      |
| `Complexity`  | `Difficult to use and understand` for many developers and administrators, who may not know how to configure LDAP clients correctly or use it securely.                                                                            |
| `Encryption`  | LDAP `does not encrypt its traffic by default`, which exposes sensitive data to potential eavesdropping and tampering. LDAPS (LDAP over SSL) or StartTLS must be used to enable encryption.                                       |
| `Injection`   | `Vulnerable to LDAP injection attacks`, where malicious users can manipulate LDAP queries and `gain unauthorised access` to data or resources. To prevent such attacks, input validation and output encoding must be implemented. |

* Commonly used for providing a central location for accessing an managing directory services. 
* LDAP enables organisations to store, manage andd secure this information in a standard way.
#### Use cases:

|**Use Case**|**Description**|
|---|---|
|`Authentication`|LDAP can be used for `central authentication`, allowing users to have single login credentials across multiple applications and systems. This is one of the most common use cases for LDAP.|
|`Authorisation`|LDAP can `manage permissions` and `access control` for network resources such as folders or files on a network share. However, this may require additional configuration or integration with protocols like Kerberos.|
|`Directory Services`|LDAP provides a way to `search`, `retrieve`, and `modify data` stored in a directory, making it helpful for managing large numbers of users and devices in a corporate network. `LDAP is based on the X.500 standard` for directory services.|
|`Synchronisation`|LDAP can be used to `keep data consistent` across multiple systems by `replicating changes` made in one directory to another.|

#### Types of LDAP
* There are 2 Implementations of LDAP: 
	1. OpenLDAP
	2. Microsoft Active Directory 
	
| **LDAP**                                                                                                                                   | **Active Directory (AD)**                                                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| A `protocol` that defines how clients and servers communicate with each other to access and manipulate data stored in a directory service. | A `directory server` that uses LDAP as one of its protocols to provide authentication, authorisation, and other services for Windows-based networks.                                                         |
| An `open and cross-platform protocol` that can be used with different types of directory servers and applications.                         | `Proprietary software` that only works with Windows-based systems and requires additional components such as DNS (Domain Name System) and Kerberos for its functionality.                                    |
| It has a `flexible and extensible schema` that allows custom attributes and object classes to be defined by administrators or developers.  | It has a `predefined schema` that follows and extends the X.500 standard with additional object classes and attributes specific to Windows environments. Modifications should be made with caution and care. |
| Supports `multiple authentication mechanisms` such as simple bind, SASL, etc.                                                              | It supports `Kerberos` as its primary authentication mechanism but also supports NTLM (NT LAN Manager) and LDAP over SSL/TLS for backward compatibility.                                                     |
#### How it works
* LDAP works on a client-server architecture.
* The clients sends an LDAP request to a server 
* This searches the directory service and returns a response to the client. 
* It is simpler and more efficient than X.500 on which it is based. 
##### LDAP request example:
1. **Session connections** - The client connects to the server via LDAP port (usually 389 or 636)
2. **Request Type** - Client specifics the operation it wants to perform, such as **bind**, **search** etc.
3. **Request Parameter** - The client provides additional information for the request such as **distinguished name (DN)** of the entry to be accessed or modified, the scope and filter of the search query. 
4. **Request ID** - The client assigns a unique identifier for each request to match it with the corresponding response from the server. 
##### Reponse Message
1. **Response Type** - The server indicates the operation that was performed to the request. 
2. **Result Code** - The server indicates whether or not the operation was successful and why
3. **Matched DN** - If applicable, the server returns the DN of the closest existing entry that matches the request. 
4. **Referral** - The server returns a URL of another server that may have more information about the request. 
5. **Response Data** - The server returns any additional data related to the response as the attributes and values of an entry that was searched or modified,
## Enumeration and Attacks
```shell
ldapsearch -H ldap://ldap.example.com:389 -D "cn=admin,dc=example,dc=com" -w secret123 -b "ou=people,dc=example,dc=com" "(mail=john.doe@example.com)"
```
1. `-H` Connect to the server on `ldap.example.com:389` 
2. `-D` Bind (authenticate) as `cn=admin,dc=example,dc=com` with password `secret123`
3. Search under the base DN `ou=people,dc=example,dc=com`
4. Use the filter `(mail=john.doe@example.com)` to find entries that have that address. 
### LDAP Injection
* Exploits web applications that use LDAP for authentication or storing user information, 
* The attacker can inject malicious code or characters into LDAP queries to alter the applications behaviours, bypass security measures and access sensitive data. 
#### Test for LDAP: 

| Input    | Description                                                                                                                                                                                                                                          |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `*`      | An asterisk `*` can `match any number of characters`.                                                                                                                                                                                                |
| `( )`    | Parentheses `( )` can `group expressions`.                                                                                                                                                                                                           |
| `\|`     | A vertical bar `\|` can perform `logical OR`.                                                                                                                                                                                                        |
| `&`      | An ampersand `&` can perform `logical AND`.                                                                                                                                                                                                          |
| `(cn=*)` | Input values that try to bypass authentication or authorisation checks by injecting conditions that `always evaluate to true` can be used. For example, `(cn=*)` or `(objectClass=*)` can be used as input values for a username or password fields. |

##### Example: 
LDAP query:
```php
(&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))
```
Injection: 
```php
$username = "*";
$password = "dummy";
(&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))
```
### NMAP
```shell
nmap -p- -sC -sV --open --min-rate=1000 10.129.204.229
```
