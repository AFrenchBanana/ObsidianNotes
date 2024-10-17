### Making a bash script 
* Should end in `.sh`
* Should start with a shebang, tells the computer what binary to run as:
	* `#!/bin/bash`

## Variables 
```bash 
variable1 = Hello
variable2 = World
# read variables:
echo $variable1 $variable2
read variable1
```
## Functions 
```shell
function_name () {
	<commands>
}
# alternate method 
function function_name{
	<commands>
}
```
Bracket should always be empty

### Special Variables 
Predefined Variables:

| Value       | use                                                |
| ----------- | -------------------------------------------------- |
| `$0`        | Name of the bash script                            |
| `$1- $9`    | The first 9 arguments in the bash script           |
| `$#`        | How many arguments were passed to the bash script  |
| `$@`        | All the arguments passed to the bash script        |
| `$$`        | Process ID of the current script                   |
| `$USER`     | Username of the user running the script            |
| `$HOSTNAME` | The host name the script is running on             |
| `$SECONDS`  | The number of seconds since the script was started |
| `$RANDOM`   | Returns a different random number each time        |
| `$LINENO`   | Returns the current line number in the bash script |

## Conditional Logic 
```bash 
if [ condition ] ; then 
	commands
else [ condition ] ; then 
	commands 
elif [ conditions ] ; then 
	commands
fi
```

## Comparison Operators
| Commands | Example Usage                                           |
| -------- | ------------------------------------------------------- |
| `-eq`    | Returns true of two numbers are equal                   |
| `-lt`    | returns true if number is less than another number      |
| `-gt`    | Returns true if a number is greater than another number |
| `==`     | Returns true if two strings are =                       |
| `!=`     | Returns true if two string are not equal                |
| `!`      | Returns True if the following expression is false       |
| `-d`     | Checks the existence of a directory                     |
| `-e`     | Checks the existence of a directory                     |
| `-r`     | Checks the existence of a file and read permission      |
| `-w`     | Checks the existence of a file and write permission     |
| `-x`     | Checks the existence of a file and execute permission   |

### For Loops
```bash
for VAR in 1 2 3
do 
<actions>
done
# alternate means
for VAR in {0..1000} ...
# increment by more than one 
for VAR in {0..1000..2}
# iterate over parameters
for VAR in file1, file2, file3 
```
### While Loops
```bash
while [condition]
do
	commandss
done
```
### Break
```bash
while true
do 
	i ++ 
	if [i == 10]
		break
	fi
done
```
### Continue 
Bypass rest of commands in loop and continue next iteration
```bash
while true
do 
	i ++ 
	if [i == 10]
		continue
	fi
done
```