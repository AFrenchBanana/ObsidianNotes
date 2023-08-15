## Protected Files 

### Hunting for  encoded Files.

```shell
for ext in $(echo ".xls .xls* .xltx .csv .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
### Hunting for #SSH Keys 

```shell
grep -rnw "PRIVATE KEY" /* 2>/dev/null | grep ":1"
```

##### Cracking Encrypted SSH Keys with #John 
```shell
ssh2john.py SSH.private > ssh.hash
```

```shell
john --wordlist=rockyou.txt ssh.hash
```

### Microsoft #Office Files
```shell
office2john.py
```
### #PDFs

```shell
pdf2john.py PDF.pdf > pdf.hash
```


## Protected Archives
Download all file extensions 
```shell
curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```

#### #Zip 
```shell
zip2john
```

#### Encrypted with #openssl
Best to run a for loop
```shell
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in <filename> -k $i 2>/dev/null| tar xz;done
```

#### Bitlocker 

May be able to get virtual hardrives 
```shell
bitlocker2john -i Backup.vhd > backup.hashes
```
##### #Hashcat 
```shell
hashcat -m 22100 backup.hash /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt -o backup.cracked
```