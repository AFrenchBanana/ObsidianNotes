## Ping
### Ping Sweep
Windows 
```cmd
for /1 %i in (1,1,254) do @ping -n 1 -w 100 (first 3 octets).%i
```
Linux 
```bash
for i in {1..255}; do ping -c 1 -W 1 (first 3 octets).$i ; done
```
## NetDiscover 
* Actively and passively scan wireless networks
* Sends out multiple ARP requests
* Queries MAC addresses against an OUI table

## NMAP

#### Timing options:
`-Tx`

| Number        | What occurs                                        |
| ------------- | -------------------------------------------------- |
| Paranoid (0)  | IDS evasion                                        |
| Paranoid (1)  | IDS evasion                                        |
| Polite(2)     | Slower scan which uses less bandwidth (10x slower) |
| Normal (3)    | Default type                                       |
| Aggressive(4) | Faster scans                                       |
| Insane (5)    | Sacrifices accuracy for speed                      |

#### Scan Types

| Flag  | Scan                                |
| ----- | ----------------------------------- |
| `-sT` | full TCP Scan (noisy option)        |
| `-sU` | UDP Scan (can have false positives) |
| `-sS` | Default Syn scan                    |

#### Output options

| Flag  | Output Option          |
| ----- | ---------------------- |
| `-oN` | Normal Output          |
| `-oX` | XML Output             |
| `-oG` | Grepable output        |
| `-oA` | Outputs to all options |



