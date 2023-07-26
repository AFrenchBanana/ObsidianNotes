### Features
Oracle Transport Network Substrate (TNS)
Often used for managing large complex databases in healthcare, finance and retail.
Built-in encryption 
By default can be managed Remotely  

### Configuration 
Files called tnsnames.ora or listener.ora
	ORACLE_HOME/network/admin 
	client side use tnsnames and server side uses listener,
Each database has a unique tnsnames.ora file.
	Contains necessary information for clients to connect to the service 
	
|**Setting**|**Description**|
|---|---|
|`DESCRIPTION`|A descriptor that provides a name for the database and its connection type.|
|`ADDRESS`|The network address of the database, which includes the hostname and port number.|
|`PROTOCOL`|The network protocol used for communication with the server|
|`PORT`|The port number used for communication with the server|
|`CONNECT_DATA`|Specifies the attributes of the connection, such as the service name or SID, protocol, and database instance identifier.|
|`INSTANCE_NAME`|The name of the database instance the client wants to connect.|
|`SERVICE_NAME`|The name of the service that the client wants to connect to.|
|`SERVER`|The type of server used for the database connection, such as dedicated or shared.|
|`USER`|The username used to authenticate with the database server.|
|`PASSWORD`|The password used to authenticate with the database server.|
|`SECURITY`|The type of security for the connection.|
|`VALIDATE_CERT`|Whether to validate the certificate using SSL/TLS.|
|`SSL_VERSION`|The version of SSL/TLS to use for the connection.|
|`CONNECT_TIMEOUT`|The time limit in seconds for the client to establish a connection to the database.|
|`RECEIVE_TIMEOUT`|The time limit in seconds for the client to receive a response from the database.|
|`SEND_TIMEOUT`|The time limit in seconds for the client to send a request to the database.|
|`SQLNET.EXPIRE_TIME`|The time limit in seconds for the client to detect a connection has failed.|
|`TRACE_LEVEL`|The level of tracing for the database connection.|
|`TRACE_DIRECTORY`|The directory where the trace files are stored.|
|`TRACE_FILE_NAME`|The name of the trace file.|
|`LOG_FILE`|The file where the log information is stored.|

### Footprinting 
#### May need t download some files: 
```bash
#!/bin/bash

sudo apt-get install libaio1 python3-dev alien python3-pip -y
git clone https://github.com/quentinhardy/odat.git
cd odat/
git submodule init
sudo submodule update
sudo apt install oracle-instantclient-basic oracle-instantclient-devel oracle-instantclient-sqlplus -y
pip3 install cx_Oracle
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor pycryptodome passlib python-libnmap
sudo pip3 install argcomplete && sudo activate-global-python-argcomplete
```


### Test Installation with odat 
```shell-session
./odat.py -h
```

### NMAP
```shell-session
sudo nmap -p1521 -sV <ip> --open
```
#### SID Brute Forcing 
```shell-session
sudo nmap -p1521 -sV <ip> --open --script oracle-sid-brute
```

### ODAT 
```shell-session
./odat.py all -s <ip>
```

### SQLplus Login 
```shell-session
sqlplus <user>/<pass>@10.129.204.235/XE;
```

"`sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory`,":
```shell-session

```

### Database Interaction
```shell-session
select table_name from all_tables;
```
```shell-session
sqlplus <user/<pass>@<ip>/XE as sysdba
```
#### Extract Password Hashes
```shell-session
select name, password from sys.user$;
```

### Shells 
Requires web server to be run and root directory 

|**OS**|**Path**|
|---|---|
|Linux|`/var/www/html`|
|Windows|`C:\inetpub\wwwroot`|

```shell-session
echo "Oracle File Upload Test" > testing.txt
./odat.py utlfile -s <ip> -d XE -U <user> -P <pass> --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```
