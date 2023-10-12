## System Information 
```bash
uname -a

uptime

timedatectl

screenfetch
```

## User Information 
```bash
w

lastlog # last logged in 

faillog # failed login attempts

cat /etc/passwd

cat /etc/group

cat /etc/sudoers

lsof -u [username] # files opened by user

id # user group names etc
```
## File and Drive information 
```bash 
df -ah # Local disk space

file [filename]

ls -alt # list files by most recently used

mount -t [name] # mount a file system 

cat /etc/mnttab # vie wlist of filesystems

cat /etc/fstab # view currently mounted file systems

lsusb # List USB devices attached 

lsof # List all open file belonging to all active processes
```

