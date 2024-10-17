### Overview Section
1. Source IPs
2. Special Considerations (over a VPN etc)
3. Type of Work done. 
4. Dates that work was done
5. Add a discalimer:
	1. "This report represents a snapshot in time during the aforementioned testing period, and Acme Consulting, I cannot attest to the state of any client-owned information assets outside of this testing window."

## Notetaking and Organisation 
### Sample Structure 
#### Attack Path 
Outline of the entire path if you gain a foothold during an external pentest.
Use Screenshots and command output to make it easier to paste into a report later.
#### Credentials 
Centralised place to keep comprised credentials and secrets
#### Findings
Subfolder for head finding then writing narrative and saving folder along with any evidence. 
#### Vulnerability Scans Research 
Take notes of research you done
#### Service Enumeration Research 
Section to take notes on which services you've investigated, failed exploitation attempts, promising vulns.
#### Web Application Research 
Note down interesting web applications found through various methods such as sub domain brute forcing. 
#### AD Enumeration Research 
What enumeration has been done and anything interesting
#### OSINT
Keep track of interesting information you've collected via OSINT.
#### Administrative Information
Project managers, Point of calls,
To do list, 
Rules of Engagment
#### Scoping information 
in-scope IP addresses
URLS
creentials
VPN, AD provided by client.
#### Activity Log
High level tracking of everything you done during the assessment. 
#### Payload log
Tracking payloads being used and a **file hash for anything uploaded**

### Logging
Log all scanning and attack attempts and keep raw tool output. 
#### [Tmux Logging](about:blank)
Terminal Logging. 
Tracks everything type into Tmux.
```shell
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

```shell
touch .tmux.conf
```

```shell
[cat .tmux.conf](<cat .tmux.conf 

# List of plugins

set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-logging'

# Initialize TMUX plugin manager (keep at bottom)
run '~/.tmux/plugins/tpm/tpm'>)
```

```shell
tmux source ~/.tmux.conf 
```

To Start Logging in the current session
`[Ctrl] + [B]` followed by `[Shift] + [P]`

Retroactive logging 
`
` and then hitting `[Alt] + [Shift] + [P]`

Add Line to .tmux.conf
```shell
set -g history-limit 50000
```
##### Screenshots 
`[Ctrl] + [B]` followed by `[Alt] + [P]`
This will then output to a file

#### Payload Logging
- IP address of the host(s)/hostname(s) where the change was made
- Timestamp of the change
- Description of the change
- Location on the host(s) where the change was made
- Name of the application or service that was tampered with
- Name of the account (if you created one) and perhaps the password in case you are required to surrender it
### Evidence 
Terminal Output for logs or screenshots 
#### Folder Structure 
Make the directories 
```shell
 mkdir -p ACME-IPT/{Admin,Deliverables,Evidence/{Findings,Scans/{Vuln,Service,Web,'AD Enumeration'},Notes,OSINT,Wireless,'Logging output','Misc Files'},Retest}
```

- `Admin`
        - Scope of Work (SoW) that you're working off of, your notes from the project kickoff meeting, status reports, vulnerability notifications, etc
- `Deliverables`
        - Folder for keeping your deliverables as you work through them. This will often be your report but can include other items such as supplemental spreadsheets and slide decks, depending on the specific client requirements.
- `Evidence`
        - Findings
        - We suggest creating a folder for each finding you plan to include in the report to keep your evidence for each finding in a container to make piecing the walkthrough together easier when you write the report.
    - Scans
        - Vulnerability scans
            - Export files from your vulnerability scanner (if applicable for the assessment type) for archiving.
        - Service Enumeration
            - Export files from tools you use to enumerate services in the target environment like Nmap, Masscan, Rumble, etc.
        - Web
            - Export files for tools such as ZAP or Burp state files, EyeWitness, Aquatone, etc.
        - AD Enumeration
            - JSON files from BloodHound, CSV files generated from PowerView or ADRecon, Ping Castle data, Snaffler log files, CrackMapExec logs, data from Impacket tools, etc.
    - Notes
        - A folder to keep your notes in.
    - OSINT
        - Any OSINT output from tools like Intelx and Maltego that doesn't fit well in your notes document.
    - Wireless
        - Optional if wireless testing is in scope, you can use this folder for output from wireless testing tools.
    - Logging output
        - Logging output from Tmux, Metasploit, and any other log output that does not fit the `Scan` subdirectories listed above.
    - Misc Files
        - Web shells, payloads, custom scripts, and any other files generated during the assessment that are relevant to the project.
- `Retest`
        - This is an optional folder if you need to return after the original assessment and retest the previously discovered findings. You may want to replicate the folder structure you used during the initial assessment in this directory to keep your retest evidence separate from your original evidence.

### Formatting and Redaction 
All personal information and credentials should be redacted
Anything that is graphic or obscene should be. 

### Screenshots
Try and use terminal output over screenshots as it is easier to redact. 
Highlight the important parts
use `<SNIP>`
`<REDACTED>`
`<PASSWORD REDACTED>`

