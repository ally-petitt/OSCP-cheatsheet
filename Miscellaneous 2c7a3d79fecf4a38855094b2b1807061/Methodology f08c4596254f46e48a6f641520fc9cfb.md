# Methodology

# SMB

If you see smb open, check for CVEs.

```bash
nmap --script smb-vuln* -Pn -p 139,445 -oN nmap/smb-vuln 192.168.96.43
```

Check null authenication, guest authentication, anonymous access, `admin: admin`, `administrator:administrator`, etc.