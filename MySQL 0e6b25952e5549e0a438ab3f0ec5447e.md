# MySQL

Default creds: `root:` (no password)

# With Creds

If there is a website running and you can't crack the admin hash, generate a new one and replace it. This is done here [Exploit](https://www.notion.so/Exploit-e9901369489d42ebadd21571fb12f61d).

[PostgresSQL](MySQL%200e6b25952e5549e0a438ab3f0ec5447e/PostgresSQL%20e498983a19fd4f41881b473aa3909607.md)

# Local Privilege Escalation

If you are able to get into MySQL on the target host, check your status with `mysql> status` if you are revealed to be root user like shown below,

![Untitled](MySQL%200e6b25952e5549e0a438ab3f0ec5447e/Untitled.png)

OR you `ps aux | grep mysql` and see this 

![Untitled](MySQL%200e6b25952e5549e0a438ab3f0ec5447e/Untitled%201.png)

You can escalate your privileges through user defined functions.Tutorial here.

[Linux Privilege Escalation - Exploiting User-Defined Functions - StefLan's Security Blog](https://steflan-security.com/linux-privilege-escalation-exploiting-user-defined-functions/)

> ***************************************************************Note: Make sure that the permissions of the raptor_udf.so and raptor_udf.o files are set to 777***************************************************************
> 

You can find **precompiled binaries** [here](https://github.com/rapid7/metasploit-framework/tree/master/data/exploits/mysql).

[MSSQL](MySQL%200e6b25952e5549e0a438ab3f0ec5447e/MSSQL%203ccca60abe454be8bee82e97c920a60a.md)

[MongoDB](MySQL%200e6b25952e5549e0a438ab3f0ec5447e/MongoDB%20e4058f55dfe84c82a4d0c265e787b50f.md)

[MySQLi](MySQL%200e6b25952e5549e0a438ab3f0ec5447e/MySQLi%2053e8d807c5504b24a0c65bfb9fc1cc33.md)