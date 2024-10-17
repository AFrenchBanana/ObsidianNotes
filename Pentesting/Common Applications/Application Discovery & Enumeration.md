# Discovery 
```shell
cat scope_list
<domain1>
<domain2>
<domain3>
...
```
## Nmap Web Discovery
```shell
nmap -sV -p 80,443,8000,8080,8180,8888,1000 --open -oA web_discovery -iL scope_list
```

## EyeWitness
* Can take XML output of Nmap and Nessus and create a report with screenshots of each web application present.
* Can also give a list of URLs with `http://` or `https://`
```shell
eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```
## AquaTone
```shell
wget https://github.com/michenriksen/aquatone/releases/download/v1.7.0/aquatone_linux_amd64_1.7.0.zip;
unzip aquatone_linux_amd64_1.7.0.zip 
```
Move to `$PATH` if you want
```shell
cat web_discovery.xml | ./aquatone -nmap
```

