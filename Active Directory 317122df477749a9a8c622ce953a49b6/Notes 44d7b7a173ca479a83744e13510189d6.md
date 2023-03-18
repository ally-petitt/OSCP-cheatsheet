# Notes:

# ****[This Active Directory Method Helped Me Pass OSCP](https://youtu.be/aZsysS4BaTs)****

### Resources

1. [https://pentest.coffee/active-directory-cheat-sheet-94e0bb9bed2](https://pentest.coffee/active-directory-cheat-sheet-94e0bb9bed2) Another cheatsheet
2. [Post-exploitation mind map](https://xmind.app/m/vQuTSG/) 
3. [https://lyethar.gitbook.io/methodology/readme/active-directory/exploitation/gmsa-password-read](https://lyethar.gitbook.io/methodology/readme/active-directory/exploitation/gmsa-password-read) More on ACL abuse

## Initial Foothold

- Focus on finding usernames and using them for AS-REP roast
- User newer Enum4Linux-NG for better SMB enumeration
- 

### Enumeration

```nasm
smbclient -L '\\IP\' -U 'Guest' -N
smbclient -L '\\IP\' -U '' -N
smbclient -L '\\IP\' -U 'Anonymous' -N
smbclient -L '\\IP\' -U 'Anonymous' -P 'Anonymou'
ldapsearch -x -h IP
```

## Decrypting powershell credentials

If you find an XML file that looks like this in Windows Active Directory and you don’t know what it means. Perhaps you thought it was Group Policy (GPP) and tried `gpp-decrypt`but it didn’t work. 

```abap
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">Your Flag is here =&gt;</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb010000009db56a0543f441469fc81aadb02945d20000000002000000000003660000c000000010000000069a026f82c590fa867556fe4495ca870000000004800000a0000000100000003b5bf64299ad06afde3fc9d6efe72d35500000002828ad79f53f3f38ceb3d8a8c41179a54dc94cab7b17ba52d0b9fc62dfd4a205f2bba2688e8e67e5cbc6d6584496d107b4307469b95eb3fdfd855abe27334a5fe32a8b35a3a0b6424081e14dc387902414000000e6e36273726b3c093bbbb4e976392a874772576d</SS>
    </Props>
  </Obj>
</Objs>
```

Well, this is a PS Credential. You can decrypt it like this 

```abap
$Credential = Import-Clixml -Path "lvetrova.xml"
$Credential.GetNetworkCredential().password
```

*****************************************************************************Note: You usually have to be logged in as the user whose session it was encrypted in for it to work.*****************************************************************************

# IP vs Hostnames

**Question:** *Is there a difference between `dir \\za.tryhackme.com\SYSVOL` and `dir \\<DC IP>\SYSVOL` and why the big fuss about DNS?*

There is quite a difference, and it boils down to the authentication method being used. When we provide the hostname, network authentication will attempt first to perform Kerberos authentication. Since Kerberos authentication uses hostnames embedded in the tickets, if we provide the IP instead, we can force the authentication type to be NTLM. While on the surface, this does not matter to us right now, it is good to understand these slight differences since they can allow you to remain more stealthy during a Red team assessment. In some instances, organisations will be monitoring for OverPass- and Pass-The-Hash Attacks. Forcing NTLM authentication is a good trick to have in the book to avoid detection in these cases.

**************************************(Copied from TryHackMe enumerating AD)**************************************

# Got stuck?

- Look for Windows Privesc instead of bloodhound
- look in program files directories and if there is anything beyond the default, look up privesc exploits for them
- When you get first access to AD, enumerate all users and groups so you know your targets.
- Offsec does acls, delegations, dcsync, golden ticket, silver ticket to rce, kerberoasting