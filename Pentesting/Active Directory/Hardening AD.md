#### Things To Document and Track

- `Naming conventions of OUs, computers, users, groups`
- `DNS, network, and DHCP configurations`
- `An intimate understanding of all GPOs and the objects that they are applied to`
- `Assignment of FSMO roles`
- `Full and current application inventory`
- `A list of all enterprise hosts and their location`
- `Any trust relationships we have with other domains or outside entities`
- `Users who have elevated permissions`

#### Things to Do 
- the organization should have a strong password policy, with a password filter that disallows the use of common words (i.e., welcome, password, names of months/days/seasons, and the company name). If possible, an enterprise password manager should be used to assist users with choosing and using complex passwords.
- Rotate passwords periodically for **all** service accounts.
- Disallow local administrator access on user workstations unless a specific business need exists.
- Disable the default `RID-500 local admin` account and create a new admin account for administration subject to LAPS password rotation.
- Implement split tiers of administration for administrative users. Too often, during an assessment, you will gain access to Domain Administrator credentials on a computer that an administrator uses for all work activities.
- Clean up privileged groups. `Does the organization need 50+ Domain/Enterprise Admins?` Restrict group membership in highly privileged groups to only those users who require this access to perform their day-to-day system administrator duties.
- Where appropriate, place accounts in the `Protected Users` group.
- Disable Kerberos delegation for administrative accounts (the Protected Users group may not do this)

### Protected User Groups
```powershell
Get-ADGroup -Identity "Protected Users" -Properties Name,Description,Members
```
- Group members can not be delegated with constrained or unconstrained delegation.
- CredSSP will not cache plaintext credentials in memory even if Allow delegating default credentials is set within Group Policy.
- Windows Digest will not cache the user's plaintext password, even if Windows Digest is enabled.
- Members cannot authenticate using NTLM authentication or use DES or RC4 keys.
- After acquiring a TGT, the user's long-term keys or plaintext credentials are not cached.
- Members cannot renew a TGT longer than the original 4-hour TTL.

### Proccesses
- Proper policies and procedures for AD asset management.
    - AD host audit, the use of asset tags, and periodic asset inventories can help ensure hosts are not lost.
- Access control policies (user account provisioning/de-provisioning), multi-factor authentication mechanisms.
- Processes for provisioning and decommissioning hosts (i.e., baseline security hardening guideline, gold images)
- AD cleanup policies
    - `Are accounts for former employees removed or just disabled?`
    - `What is the process for removing stale records from AD?`
    - Processes for decommissioning legacy operating systems/services (i.e., proper uninstallation of Exchange when migrating to 0365).
    - Schedule for User, groups, and hosts audit.

### Technology 
- Run tools such as BloodHound, PingCastle, and Grouper periodically to identify AD misconfigurations.
- Ensure that administrators are not storing passwords in the AD account description field.
- Review SYSVOL for scripts containing passwords and other sensitive data.
- Avoid the use of "normal" service accounts, utilizing Group Managed (gMSA) and Managed Service Accounts (MSA) where ever possible to mitigate the risk of Kerberoasting.
- Disable Unconstrained Delegation wherever possible.
- Prevent direct access to Domain Controllers through the use of hardened jump hosts.
- Consider setting the `ms-DS-MachineAccountQuota` attribute to `0`, which disallows users from adding machine accounts and can prevent several attacks such as the noPac attack and Resource-Based Constrained Delegation (RBCD)
- Disable the print spooler service wherever possible to prevent several attacks
- Disable NTLM authentication for Domain Controllers if possible
- Use Extended Protection for Authentication along with enabling Require SSL only to allow HTTPS connections for the Certificate Authority Web Enrollment and Certificate Enrollment Web Service services
- Enable SMB signing and LDAP signing
- Take steps to prevent enumeration with tools like BloodHound
- Ideally, perform quarterly penetration tests/AD security assessments, but if budget constraints exist, these should be performed annually at the very least.
- Test backups for validity and review/practice disaster recovery plans.
- Enable the restriction of anonymous access and prevent null session enumeration by setting the `RestrictNullSessAccess` registry key to `1` to restrict null session access to unauthenticated users.
### AD Snapshot
[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) is part of the Sysinternal Suite and is described as:

"An advanced Active Directory (AD) viewer and editor. You can use AD Explorer to navigate an AD database easily, define favorite locations, view object properties, and attributes without opening dialog boxes, edit permissions, view an object's schema, and execute sophisticated searches that you can save and re-execute."

### Ping Castle
[PingCastle](https://www.pingcastle.com/documentation/) is a powerful tool that evaluates the security posture of an AD environment and provides us the results in several different maps and graphs.

### Group3r
[Group3r](https://github.com/Group3r/Group3r) is a tool purpose-built to find vulnerabilities in Active Directory associated Group Policy. Group3r must be run from a domain-joined host with a domain user (it does not need to be an administrator), or in the context of a domain user (i.e., using `runas /netonly`).
```cmd
group3r.exe -f <filepath-name.log> 
```
### AD Recon
```powershell
.\ADRecon.ps1
```

