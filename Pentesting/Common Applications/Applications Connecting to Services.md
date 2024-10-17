## ELF executable Examination 
```shell
gdb ./<binary>
```
* Disassemble the main function 
```assembly
set disassembly-flavor intel
```
```assembly
disas main
```
```assembly
run
```
* Look for call functions
* Inspect the break 
```assembly
 b *0x5555555551b0
```
## DLL File Examination 
```powershell
Get-FileMetaData <dll>
```
* You can view the source code with [dnSpy](https://github.com/dnSpy/dnSpy)
