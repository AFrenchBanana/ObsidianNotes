## IIS 
* Internet Information Services (IIS) is a general-purpose web server. 
### What is IIS Tilde Enumeration 
* A technique utilised to uncover hidden files, directories and short file names (`8.3 format`)
* When a file or folder is generated a short filename in the `8.3` format. 
* This consists of eight characters for the file name, a period and 3 characters for the extension. 
* These short file names can grant access to their corresponding files and folders even if they were meant to be hidden or inaccessible. 
* The tilde character `~`, followed by a sequence number, signifies a short file name in a URL. 
```http
http://example.com/~a
http://example.com/~b
http://example.com/~c
```
* When the server contains a hidden directory named Secret Documents:
* a request is sent with `~s` and then this replies with a `200 Ok`
```http
http://example.com/~sec
http://example.com/~secr
http://example.com/~secre
```
* Then when you send `se` it returns another 200
* and so one til `secretdoc` is made. 
### Exploitation
```shell
nmap -p- -sV -sC --open 10.129.224.91
```
Look for Microsoft IIS in service verisons.
#### Tilde Enumeration Using IIS ShortName Scanner
[Link to scanner](https://github.com/irsdl/IIS-ShortName-Scanner)
* Need oracle and java installed
```shell
java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/
```
If files are found you may need to brute force `GET` requests for this. 
* They may be longer than 8 characters allowed by `8.3`
#### Generate Word List
```shell
 egrep -r ^<foundword_no_extension> /usr/share/wordlists/ | sed 's/^[^:]*://' > /tmp/list.txt
```

| **Command Part**    | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                     |                  |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| `egrep -r ^transf`  | The `egrep` command is used to search for lines containing a specific pattern in the input files. The `-r` flag indicates a recursive search through directories. The `^transf` pattern matches any line that starts with "transf". The output of this command will be lines that begin with "transf" along with their source file names.                                                                                           |                  |
| `\|`                | The pipe symbol (`\|`) is used to pass the output of the first command (`egrep`) to the second command (`sed`). In this case, the lines starting with "transf" and their file names will be the input for the `sed` command.                                                                                                                                                                                                        |                  |
| `sed 's/^[^:]*://'` | The `sed` command is used to perform a find-and-replace operation on its input (in this case, the output of `egrep`). The `'s/^[^:]*://'` expression tells `sed` to find any sequence of characters at the beginning of a line (`^`) up to the first colon (`:`), and replace them with nothing (effectively removing the matched text). The result will be the lines starting with "transf" but without the file names and colons. |                  |
| `> /tmp/list.txt`   | The greater-than symbol (`>`) is used to redirect the output of the entire command (i.e., the modified lines) to a new file named `/tmp/list.txt`.                                                                                                                                                                                                                                                                                  | |

```shell
gobuster dir -u http://10.129.204.231/ -w /tmp/list.txt -x <extension>
```
