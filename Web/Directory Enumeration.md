## What to Look for
1. Listable directories 
2. Login Pages
3. Default Pages
4. Special Pages 
5. Nested sites

## Manual Enumeration 
### Robots.txt
* `sitemap:` - Directs the engine to a list of websites pages used for indexing 
* `Disallow:` - Would allow everything to be indexed
* `Disallow: /` -  Would disallow everything to be indexed
* `Disallow: /wp-admin/` - Prevents search engines indexing wp site
* `Disallow: /?s=` - Prevents search parameter crawling 
### Sitemap.xml
Used to quickly index URLs
Format should be 
```XML
<url>https://example.com</url>
```

## Automated tools 
### Dirb
| Command                 | Description                |
| ----------------------- | -------------------------- |
| `dirb [url]`            | Default scan               |
| `dirb [url] [wordlist]` | default scan with wordlist |
| `-w`                    | Ignore warnings and keep scanning                         |
| `-N [code]`             |Ignore responses with a specified HTTP code                     |
| `-t`                    |dont add / to urls|
| `-i`                    |    case insensitive search|
| `-s`                    |silent mode no output                            |
| `-o [file]`             |output to a file      |
| `-r`                    |don't scan recursively             |
| `-R`                    |stop and ask on each directory|
| `-a`                    |specify USER_AGENT                            |
| `-u [username:password]`|username and password                      |

### GoBuster 
| Command                                            | Description |
| -------------------------------------------------- | ----------- |
| `gobuster dir -u [url] -w [wordlist]`              | Default scan            |
| `gobuster dir -u [url] -w [wordlist] -x php,html ` |Scan for php and html files             |
| `-x html,php,css`                                  |list file extensions             |
| `-o [file]`                                        |write results to file             |
| `-U [username] -p [password]`                      |username and password             |
| `-v`                                               |verbose|
| `-q`                                               |quiet mode             |
| `-t [number]`                                      |Number of threads. Use <10 to reduce noise             |
| `--delay [number]`|    Add a delay between requests       | 

### Ffuf
| **Command**                                                                                                                                                     | **Description**          |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| `ffuf -h`                                                                                                                                                       | ffuf help                |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ`                                                                                                       | Directory Fuzzing        |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/indexFUZZ`                                                                                                  | Extension Fuzzing        |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php`                                                                                              | Page Fuzzing             |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v`                                                              | Recursive Fuzzing        |
| `ffuf -w wordlist.txt:FUZZ -u https://FUZZ.hackthebox.eu/`                                                                                                      | Sub-domain Fuzzing       |
| `ffuf -w wordlist.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs xxx`                                                                     | VHost Fuzzing            |
| `ffuf -w wordlist.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx`                                                                   | Parameter Fuzzing - GET  |
| `ffuf -w wordlist.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx` | Parameter Fuzzing - POST |
| `ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx`       | Value Fuzzing            |

## OSINT based subdomain enumeration 
### Assestfinder
Pulls from a number of sources to identify domains and subdomains related to target.
### Amass
Designed to perform external asset discovery 
Will use techniques such as DNS enumeration, brute forcing, reverse DNS sweeping, zone transfers and web scraping.
