# FTPS

FTP over TLS, aka FTPS, requires a different cli than `ftp`. You can use `lftp` like this

```jsx
lftp
lftp :~> set ftp:ssl-force true
lftp :~> set ssl:verify-certificate no
lftp :~> connect 10.10.10.208
lftp 10.10.10.208:~> login                       
Usage: login <user|URL> [<pass>]
lftp 10.10.10.208:~> login username Password

# or this
lftp               
lftp :~> set ftps:initial-prot ""
lftp :~> set ftp:ssl-force true
lftp :~> set ftp:ssl-protect-data true
lftp :~> open ftps://billy:21
lftp billy:~> user admin admin
lftp admin@billy:~> lcd /
lcd ok, local cwd=/
```

You can also connect via filezilla. If you don’t have it installed,

```jsx
apt update
apt get filezilla
filezilla # run GUI from command line
```

Then, connect via Filezilla which uses ftps by default.