#nfs

[Network File System](https://en.wikipedia.org/wiki/Network_File_System)
Same Purpose as #SMB 

|**Version**|**Features**|
|---|---|
|`NFSv2`|It is older but is supported by many systems and was initially operated entirely over UDP.|
|`NFSv3`|It has more features, including variable file size and better error reporting, but is not fully compatible with NFSv2 clients.|
|`NFSv4`|It includes Kerberos, works through firewalls and on the Internet, no longer requires portmappers, supports ACLs, applies state-based operations, and provides performance improvements and high security. It is also the first version to have a stateful protocol.|

Based on [ONC-RPC/SUN-RPC](https://en.wikipedia.org/wiki/Sun_RPC) on port 111 (TCP + UDP)
Used [External Data Representation](https://en.wikipedia.org/wiki/External_Data_Representation)
No authentication / authorisation, uses #RPC options

Configuration stored in[/etc/exports](https://manpages.ubuntu.com/manpages/trusty/man5/exports.5.html)

**Settings**

|**Option**|**Description**|
|---|---|
|`rw`|Read and write permissions.|
|`ro`|Read only permissions.|
|`sync`|Synchronous data transfer. (A bit slower)|
|`async`|Asynchronous data transfer. (A bit faster)|
|`secure`|Ports above 1024 will not be used.|
|`insecure`|Ports above 1024 will be used.|
|`no_subtree_check`|This option disables the checking of subdirectory trees.|
|`root_squash`|Assigns all permissions to files of root UID/GID 0 to the UID/GID of anonymous, which prevents `root` from accessing files on an NFS mount.|

**Dangerous Settings** 

|**Option**|**Description**|
|---|---|
|`rw`|Read and write permissions.|
|`insecure`|Ports above 1024 will be used.|
|`nohide`|If another file system was mounted below an exported directory, this directory is exported by its own exports entry.|
|`no_root_squash`|All files created by root are kept with the UID/GID 0.|

**nmap** 
```shell-session
sudo nmap <ip> -p111,2049 -sV -sC

sudo nmap --script nfs* <ip> -sV -p111,2049
```

**Show available Shares**

```shell-session
showmount -e <ip>
```

**Mount NFS Share**

```shell-session
mkdir target-NFS
sudo mount -t nfs <ip>:/ ./target-NFS/ -o nolock
cd target-NFS
tree .
```

**List contents**
```shell-session
ls -l mnt/nfs/

ls -n mnt/nfs/
```

**Unmount** 

```shell-session
sudo umount ./target-NFS
```
