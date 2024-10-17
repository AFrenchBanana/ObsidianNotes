## What is PowerShell
* Command-line shell, scripting language and automation platform and configuration management framework rolled into one. 
* Designed as a task engine that uses cmdlets to wrap tasks that people need to do. 
* Commands can be run on local or remote machines 
### Shared features with other shells
* Built-in help system
* PipeLine
* Aliases
* Tab completion, command prediction and History.
### How it differs
* Operates on objects over texts
* Uses cmdlets
* cmdlets typically take object input and return objects. 
	* Core cmdlets are built into .NET Core and are open source. 
	* Can build your own cmdlets in .NET Core or Powershell
## Finding Cmdlets
* Structured as Verb-noun
* Popular verbs:
	* Get, Remove, New, Update, Add, Set
* Nouns are often hierarchical
	* `Get-ADComputer`
	* `Get-ADComputerServiceAccounts`
### Searching for cmdlets
* List all cmdlets
	* `Get-Command *`
* List all cmdlets and using the "Get" verb
	* `Get-Command -Verb Get`
	* `Get-Command Get-*`
* List all cmdlets and applications that contain a specific word 
	* `Get-Command "*Service*"`
	* `Get-Command "*-AD*"

## CMD to PowerShell
* Programs used in CMD also with in PowerShell
* Common commands used in CMD are aliased as PowerShell cmdlets. 
	* `copy` has alias for `Copy-Item`
## Functions
* Declared with the word `function`
```powershell
# returns the version of powershell being used
function Get-Version { 
	$PSVersionTable.PSVersion
	}
```
Can get naming issues if the function is named the same as a cmdlets
## Variables
* Default value of all variables in PowerShell `$null`. 
* Need to make a statement to create a new variable
* Get a list of variables with `Get-Variable`
```PowerShell
$Variable1 = "a"
```

## Command Line Arguments
Easiest way to get command line argument is to use `$args` where `args[0]` will be the first argument provided, 
```PowerShell
$param = $args[0]
Write-Host $param
```
Results will show 
```PowerShell
Cmd:>test.ps1 foo
foo
```
## Conditional Logic 
* PowerShell scripts support logic such as if, elseif and else
```PowerShell
if ($Variable -ew $true) {
# do 1
}
elseif ($Variable -ew $false) {
# Do 2
}
else {
# Do 3
}
```
### Comparison Operations
- `-eq`, `-ieq`, `-ceq` = equals
- `-ne`, `-ine`, `-cne` = not equals
- `-gt`, `-igt`, `-cne` = greater than
- `-ge`, `-ige`, -`-cge` = greater than or equal
- `-lt`, `-ilt`, `-clt` = less than 
- `-le`, `-ile`, `-cle` = less than or equal
#### Matching 
`-like`, `-ilike`, `-clike` - string matches wildcard pattern
`-not like`, `-inotlike`, `-cnotlike` - string does not match pattern
`-match`, `-inotmatch`, `-cnotmatch` - string does not match 
#### Replacement 
`-replace`, `-ireplace`, `-creplace` - Replaces strings matching a regex pattern
#### Containment 
`-contains`, `-icontains`, `-ccontains` - Contains a value
`-notcontains`, `-isnotcontains`, -`cnotcontains` - collection does not contains
`-in` value is not a collection 
`-notin` value is not in a collection
#### Type
`-is` - both objects are the same
`-isnot` - the objects are not the same

### Loops
#### For 
Displays 10 multiples of 10 
```PowerShell
1..10 | Foreach-object {
	Writ-Host (10 * $_)
}
```
#### ForEach Object 
```PowerShell
$Profile_Folders = GetChildItem $env:USERPROFILES
$Profile_Folders | ForEach-Object {$_.FullName}
ForEach-Object  -InputObject $Profile_Folders -Process {$_.FullName}
```
#### While 
While, Do-While and Do-Until
```PowerShell
# Do While 
$i = 1
Do {
	$i 
	$i ++
}
While (*%i =le 10)

# Do until 
$i = 1
Do {
	$i 
	$i ++ 
}
Until ($i -gt 10)

# While
$i = 1
While ($i -le 10){
	$i
	$i ++ 
}
```
#### Break 
```PowerShell
$i = 1
While ($true){ 
	$i
	$i ++ 
	if ($i - gt 10){
		break
	}
}
```