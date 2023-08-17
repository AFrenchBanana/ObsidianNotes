## What is Domain trust 
Integrate [trusts](https://social.technet.microsoft.com/wiki/contents/articles/50969.active-directory-forest-trust-attention-points.aspx) between domains to migrate AD environments. 
Used to establish forest-forest or domain-domain authentication. 
May Allow one way or two way communication. 
### Types of Trust
- `Parent-child`: Two or more domains within the same forest. The child domain has a two-way transitive trust with the parent domain, meaning that users in the child domain `corp.inlanefreight.local` could authenticate into the parent domain `inlanefreight.local`, and vice-versa.
- `Cross-link`: A trust between child domains to speed up authentication.
- `External`: A non-transitive trust between two separate domains in separate forests which are not already joined by a forest trust. This type of trust utilizes [SID filtering](https://www.serverbrain.org/active-directory-2008/sid-history-and-sid-filtering.html) or filters out authentication requests (by SID) not from the trusted domain.
- `Tree-root`: A two-way transitive trust between a forest root domain and a new tree root domain. They are created by design when you set up a new tree root domain within a forest.
- `Forest`: A transitive trust between two forest root domains.
- [ESAE](https://docs.microsoft.com/en-us/security/compass/esae-retirement): A bastion forest used to manage Active Directory.
*Trusts can also be transitive or non-transitive*
- `transitive` trust means that trust is extended to objects that the child domain trusts. For example, let's say we have three domains. In a transitive relationship, if `Domain A` has a trust with `Domain B`, and `Domain B` has a `transitive` trust with `Domain C`, then `Domain A` will automatically trust `Domain C`.
- In a `non-transitive trust`, the child domain itself is the only one trusted.
*trusts can be set up in two directions*
- `One-way trust`: Users in a `trusted` domain can access resources in a trusting domain, not vice-versa.
- `Bidirectional trust`: Users from both trusting domains can access resources in the other domain. For example, in a bidirectional trust between `INLANEFREIGHT.LOCAL` and `FREIGHTLOGISTICS.LOCAL`, users in `INLANEFREIGHT.LOCAL` would be able to access resources in `FREIGHTLOGISTICS.LOCAL`, and vice-versa.
![[Domain Trust.png]]
## Enumerating Trust Relationships
[Get-ADTrust](https://learn.microsoft.com/en-us/powershell/module/activedirectory/get-adtrust?view=windowsserver2022-ps) cmdlet
```powershell
Import-Module activedirectory
```

```powershell
Get-ADTrust -Filter *
```
#### PowerView
```powershell
Get-DomainTrust 
```

```powershell
Get-DomainTrustMapping
```
##### Check Users in Child Domain 
```powershell
Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
```
#### netdom
##### query domain trust 
```cmd
netdom query /domain:inlanefreight.local trust
```

##### Query DCs
```cmd
netdom query /domain:inlanefreight.local dc
```
##### Query Work Stations and servers
```cmd
netdom query /domain:inlanefreight.local workstation
```
