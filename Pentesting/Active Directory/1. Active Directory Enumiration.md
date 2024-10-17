### External reconnaissance?
- Validating information provided to you in the scoping document from the client
- Ensuring you are taking actions against the appropriate scope when working remotely
- Looking for any information that is publicly accessible that can affect the outcome of your test, such as leaked credentials
#### What to look for: 

|**Data Point**|**Description**|
|---|---|
|`IP Space`|Valid ASN for our target, netblocks in use for the organization's public-facing infrastructure, cloud presence and the hosting providers, DNS record entries, etc.|
|`Domain Information`|Based on IP data, DNS, and site registrations. Who administers the domain? Are there any subdomains tied to our target? Are there any publicly accessible domain services present? (Mailservers, DNS, Websites, VPN portals, etc.) Can we determine what kind of defenses are in place? (SIEM, AV, IPS/IDS in use, etc.)|
|`Schema Format`|Can we discover the organization's email accounts, AD usernames, and even password policies? Anything that will give us information we can use to build a valid username list to test external-facing services for password spraying, credential stuffing, brute forcing, etc.|
|`Data Disclosures`|For data disclosures we will be looking for publicly accessible files ( .pdf, .ppt, .docx, .xlsx, etc. ) for any information that helps shed light on the target. For example, any published files that contain `intranet` site listings, user metadata, shares, or other critical software or hardware in the environment (credentials pushed to a public GitHub repo, the internal AD username format in the metadata of a PDF, for example.)|
|`Breach Data`|Any publicly released usernames, passwords, or other critical information that can help an attacker gain a foothold.|

#### Where to look: 

|**Resource**|**Examples**|
|---|---|
|`ASN / IP registrars`|[IANA](https://www.iana.org/), [arin](https://www.arin.net/) for searching the Americas, [RIPE](https://www.ripe.net/) for searching in Europe, [BGP Toolkit](https://bgp.he.net/)|
|`Domain Registrars & DNS`|[Domaintools](https://www.domaintools.com/), [PTRArchive](http://ptrarchive.com/), [ICANN](https://lookup.icann.org/lookup), manual DNS record requests against the domain in question or against well known DNS servers, such as `8.8.8.8`.|
|`Social Media`|Searching Linkedin, Twitter, Facebook, your region's major social media sites, news articles, and any relevant info you can find about the organization.|
|`Public-Facing Company Websites`|Often, the public website for a corporation will have relevant info embedded. News articles, embedded documents, and the "About Us" and "Contact Us" pages can also be gold mines.|
|`Cloud & Dev Storage Spaces`|[GitHub](https://github.com/), [AWS S3 buckets & Azure Blog storage containers](https://grayhatwarfare.com/), [Google searches using "Dorks"](https://www.exploit-db.com/google-hacking-database)|
|`Breach Data Sources`|[HaveIBeenPwned](https://haveibeenpwned.com/) to determine if any corporate email accounts appear in public breach data, [Dehashed](https://www.dehashed.com/) to search for corporate emails with cleartext passwords or hashes we can try to crack offline. We can then try these passwords against any exposed login portals (Citrix, RDS, OWA, 0365, VPN, VMware Horizon, custom applications, etc.) that may use AD authentication.|

##### Hunting for files and emails
inurl:
filetype:
intexxt

## Initial Enumeration of the Domain

### Key Data Points

|**Data Point**|**Description**|
|---|---|
|`AD Users`|We are trying to enumerate valid user accounts we can target for password spraying.|
|`AD Joined Computers`|Key Computers include Domain Controllers, file servers, SQL servers, web servers, Exchange mail servers, database servers, etc.|
|`Key Services`|Kerberos, NetBIOS, LDAP, DNS|
|`Vulnerable Hosts and Services`|Anything that can be a quick win. ( a.k.a an easy host to exploit and gain a foothold)|

### Passive
#### Identifying Hosts
Wireshark 
TCP dump 
Responder
```bash
sudo responder -I ens224 -A 
```
### Active
[Fping](https://fping.org/)
```shell
fping -asgq 172.16.5.0/23
```
-a: targets alive
-s: print stats
-g: generate a target list
-q: not show per target results

NMAP
```bash
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```

### Username Enumeration
#### [Kerbrute](https://github.com/ropnop/kerbrute)
Often stealthy as kerberoos pre-auth failures often don't trigger logs or alerts. 
[Statistically likely names:](https://github.com/insidetrust/statistically-likely-usernames)

```shell
sudo git clone https://github.com/ropnop/kerbrute.git
```
```shell
sudo make all
```
```shell
./kerbrute_linux_amd64 
```

Move binary to path if want?

```shell
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

### Identify Potential vulnerabilities
