## Command Line 
#### Logging in
```shell
mysql -u root -p
```

```shell
mysql -u root -p<password>
```

```shell
mysql -u root -h docker.hackthebox.eu -P 3306 -p 
```
### Interacting with tables and databases
#### Create a database
```mysql
CREATE DATABASE users;
```

```mysql
SHOW DATABASES;
```

#### Tables
##### Create a table
```MySQL
CREATE TABLE logins (
    id INT,
    username VARCHAR(100),
    password VARCHAR(100),
    date_of_joining DATETIME
    );
```

```MYSQL
SHOW TABLES;
```
##### Describe
List tables structure 
```MySQL
DESCRIBE logins;
```
##### Table Properties 
`not null`means a entry is never blank 
```mysql
username VARCHAR(100) UNIQUE NOT NULL,
```
`Auto increments` the value by 1
```Mysql
id INT NOT NULL AUTO_INCREMENT,
```
`default` specify the default value 
```MySQL
date_of_joining DATETIME DEFAULT NOW(),
```
###### Example 
```mysql
CREATE TABLE logins (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(100) NOT NULL,
    date_of_joining DATETIME DEFAULT NOW(),
    PRIMARY KEY (id)
    );
```
### Statements 
#### [Insert](https://dev.mysql.com/doc/refman/8.0/en/insert.html) 
Used to add new records to a given table
```mysql
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);
```

```mysql
INSERT INTO table_name(column2, column3, ...) VALUES (column2_value, column3_value, ...);
```
#### [Select](https://dev.mysql.com/doc/refman/8.0/en/drop-table.html) 
```mysql
SELECT * FROM table_name;
```
#### [Drop](https://dev.mysql.com/doc/refman/8.0/en/drop-table.html) 
```mysql
DROP TABLE logins;
```
#### [Alter](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html)
add new column 
```mysql
ALTER TABLE logins ADD newColumn INT;
```
rename a column 
```mysql
ALTER TABLE logins RENAME COLUMN newColumn TO oldColumn;
```
change columns datatype 
```mysql
ALTER TABLE logins MODIFY oldColumn DATE;
```
drop a column 
```mysql
ALTER TABLE logins DROP oldColumn;
```
#### [Update](https://dev.mysql.com/doc/refman/8.0/en/update.html)
Update specific records in a table 
```mysql
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;
```
example 
```mysql 
UPDATE logins SET password = 'change_password' WHERE id > 1;
```
### Query Results 
#### Sorting Results 
[ORDERBY](https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html) 
```mysql 
SELECT * FROM logins ORDER BY password;
```
Done in ascending order by default. Can add `ASC` or `DESC` to the end

```mysql
SELECT * FROM logins ORDER BY password DESC, id ASC;
```
#### [LIMIT](https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html) results 
Show only certain number of results
```mysql
SELECT * FROM logins LIMIT 2;
```
Limit with an offset (starts from 0)
```mysql
SELECT * FROM logins LIMIT 1, 2;
```
#### [Where](https://dev.mysql.com/doc/refman/8.0/en/where-optimization.html)
```mysql
SELECT * FROM table_name WHERE <condition>;
```
Examples:
```mysql
SELECT * FROM logins WHERE id > 1;

SELECT * FROM logins where username = 'admin';
```
#### [Like](https://dev.mysql.com/doc/refman/8.0/en/pattern-matching.html)
Wildcard is `%`
```mysql
SELECT * FROM logins WHERE username LIKE 'admin%';
```
Match Character lengths
```mysql
SELECT * FROM logins WHERE username like '___';
```
### SQL Operators 
#### AND operator 
```mysql 
condition1 AND condition2
```
example
```mysql
SELECT 1 = 1 AND 'test' = 'test';
```
##### Symbol Operator 
```mysql 
SELECT 1 = 1 && 'test' = 'abc';
```
#### OR Operator 
```mysql
SELECT 1 = 1 OR 'test' = 'abc';
```
##### Symbol Operator 
```mysql 
SELECT 1 = 1 || 'test' = 'abc';
```
#### NOT operators 
```mysql 
SELECT NOT 1 = 1;
```
##### Symbol Operator 
```mysql 
SELECT 1 != 1;
```
#### Common Operators 
- Division (`/`),
- Multiplication (`*`),
- and Modulus (`%`)
- Addition (`+`) and subtraction (`-`)
- Comparison (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
- NOT (`!`)
- AND (`&&`)
- OR (`||`)