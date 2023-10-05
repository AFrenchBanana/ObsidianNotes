**Network Basic Input Output System**
* Allows applications and devices on a local area network to transmit data and communicate with network hardware

* Name is comprised of a 15-character ASCII string which denotes the device name and a 1-character ASCII string which denotes the service or name record type. 

* Used to identify networks over TCP/IP

* For 2 devices to start a session, one client must call another client over TCP port 139. This is known as 'NBT over IP'

## Enumeration commands
Discover windows/samba servers on the subnet and there MAC addresses and NetBIOS name.
```shell
nbtscan -r [IP/24]
```
Enumerate windows and samba 
```bash
enum4linux -a [ip]
```
```shell
nmblookup -A [ip]
```
