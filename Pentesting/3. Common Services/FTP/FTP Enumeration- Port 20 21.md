### [Commands](https://www.smartfile.com/blog/the-ultimate-ftp-commands-list/)

| **Commands** | **Description**                                                                                                                        |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `connect`    | Sets the remote host, and optionally the port, for file transfers.                                                                     |
| `get`        | Transfers a file or set of files from the remote host to the local host.                                                               |
| `put`        | Transfers a file or set of files from the local host onto the remote host.                                                             |
| `quit`       | Exits tftp.                                                                                                                            |
| `status`     | Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on. |
| `verbose`    | Turns verbose mode, which displays additional information during file transfer, on or off.                                             |
| `ls -a`      | List all files                                                                                                                         |
| `binary`     | Set transmission to binary                                                                                                             |
| `ascii`      | set transmission to ascii                                                                                                              |
| `bye`|exit                                                                                                                                        |

Clear Text Protocols
Anonymous login?
TFTP
	Trivial File Transfer Protocol
	Simpler, yet does not provide user authentication
	Uses #UDP
#vsFTPd
```shell
cat /etc/vsftpd.conf | grep -v "#"
```
```shell
cat /etc/ftpusers
```

### Settings 

|**Setting**|**Description**|
|---|---|
|`listen=NO`|Run from inetd or as a standalone daemon?|
|`listen_ipv6=YES`|Listen on IPv6 ?|
|`anonymous_enable=NO`|Enable Anonymous access?|
|`local_enable=YES`|Allow local users to login?|
|`dirmessage_enable=YES`|Display active directory messages when users go into certain directories?|
|`use_localtime=YES`|Use local time?|
|`xferlog_enable=YES`|Activate logging of uploads/downloads?|
|`connect_from_port_20=YES`|Connect from port 20?|
|`secure_chroot_dir=/var/run/vsftpd/empty`|Name of an empty directory|
|`pam_service_name=vsftpd`|This string is the name of the PAM service vsftpd will use.|
|`rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`|The last three options specify the location of the RSA certificate to use for SSL encrypted connections.|
|`rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key`||
|`ssl_enable=NO`||
|   |   |
|---|---|
|`dirmessage_enable=YES`|Show a message when they first enter a new directory?|
|`chown_uploads=YES`|Change ownership of anonymously uploaded files?|
|`chown_username=username`|User who is given ownership of anonymously uploaded files.|
|`local_enable=YES`|Enable local users to login?|
|`chroot_local_user=YES`|Place local users into their home directory?|
|`chroot_list_enable=YES`|Use a list of local users that will be placed in their home directory?|

|**Setting**|**Description**|
|---|---|
|`hide_ids=YES`|All user and group information in directory listings will be displayed as "ftp".|
|`ls_recurse_enable=YES`|Allows the use of recurse listings.|

## Dangerous Settings 

|**Setting**|**Description**|
|---|---|
|`anonymous_enable=YES`|Allowing anonymous login?|
|`anon_upload_enable=YES`|Allowing anonymous to upload files?|
|`anon_mkdir_write_enable=YES`|Allowing anonymous to create new directories?|
|`no_anon_password=YES`|Do not ask anonymous for password?|
|`anon_root=/home/username/ftp`|Directory for anonymous.|
|`write_enable=YES`|Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?|

Debug and Trace command?
Recursive Listing 
	`ls -R`
Download all files
May cause alarm bells?
```shell
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136

tree . 
```

### NMAP

```shell
sudo nmap -sV -p21 -sC -A 10.129.14.136
```
```shell
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace
```
```shell
nmap --script=ftp-* -p 21 [ip]
```
## Interaction
```shell
nc -nv 10.129.14.136 21
```
```shell
telnet 10.129.14.136 21
```
```shell
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```
