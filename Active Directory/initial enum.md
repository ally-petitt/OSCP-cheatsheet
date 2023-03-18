# initial enum

# Low Hanging fruits

## PowerUp

```nasm
Invoke-AllChecks
```

- You can follow this up with WinPEAS if you find nothing
- Check Program Files for vulnerable applications
- Services that are running
- Check for scheduled tasks
- Unquoted service paths

# Others:

## Ldap

Remote LDAP enumeration

```abap
ldapsearch -H ldap://support.htb/ -D 'support\ldap' -x -b "CN=Users,DC=SUPPORT,DC=HTB" -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz'
```

Make sure to ************************************check descriptions************************************ of users because sometimes, they contain passwords.

Local ldap enumeration with powershell

```jsx
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

$PDC = "xor-dc01.xor.com"

$SearchString = "LDAP://"

$SearchString += $PDC + "/"

$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"

$SearchString += $DistinguishedName

$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)

$objDomain = New-Object System.DirectoryServices.DirectoryEntry($SearchString,"xor.com\daisy","XorPasswordIsDead17")

$Searcher.SearchRoot = $objDomain

#can use filter here
#$Searcher.filter="samAccountType=805306368"

$Result = $Searcher.FindAll()

Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }
    
    Write-Host "------------------------"
}
```

### DNS Enum

Try for zone transfer

```abap
$ adidnsdump -u icorp\\testuser --print-zones icorp-dc.internal.corp #https://github.com/dirkjanm/adidnsdump.git
Password: 
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Found 2 domain DNS zones:
    internal.corp
    RootDNSServers
$ dig axfr <domain name> @<nameserver>
```

# User Enum

If you have access to SMB, even if there aren’t any shares, you can get the users like this.

```bash
cme smb 192.168.94.175 --users
net rpc group members 'Domain Users' -W vault.offsec -l 192.168.49.141 -U 'guest'
```

![Untitled](initial%20enum%206b12b03b3bb14257a8d2b55de723d2a9/Untitled.png)

You can also use Kerbrute

```bash
go run main.go  userenum --dc dc /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -d hutch.offsec| tee /root/pg/practice/hutch/nmap/kerbrute.out
```

Smartbrute is a python tool that you can use to password spray and brute force credentials also. It’s a bit clunky but worth noting.

Also NMAP

```bash
nmap -p88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='vault.offsec',userdb='/usr/share/seclists/Usernames/xato-net-10-million-usernames-dup.txt" 192.168.141.172
```

## Password Spray

If you're using the hash of a local administrator or system machine account, you might need to add the flags `--local-auth`.

![Untitled](initial%20enum%206b12b03b3bb14257a8d2b55de723d2a9/Untitled%201.png)

# NTLM Credential theft through writable smb share

See details on how to exploit with an scf file here: [Attempt #1: NTLM Theft via SCF file in SMB](https://www.notion.so/Attempt-1-NTLM-Theft-via-SCF-file-in-SMB-6eabdea0657c4c10a4a316f11cb54fed)