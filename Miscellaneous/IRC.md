# IRC

[194,6667,6660-7000 - Pentesting IRC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-irc)

## Connect

To connect to IRC via netcat, you must enter your username and nickname really quickly, otherwise you might be kicked out.

```jsx
$nc -nv -C 192.168.115.44 7000
USER adler 0 * adler 
NICK adler
```

Here is an example of getting kicked out:

![Untitled](IRC%206bf6890d9696470eb65b614ad000a84a/Untitled.png)

Here is an example of successful login with quickly pasting in the creds

![Untitled](IRC%206bf6890d9696470eb65b614ad000a84a/Untitled%201.png)

## Enum

You can do a brute force attack with nmap. 

```jsx
nmap -sV --script irc-brute,irc-sasl-brute --script-args userdb=./users.txt,passdb=/usr/share/wordlists/rockyou.txt -p7000 192.168.115.44
```

Here are some of the basic commands as shown by hacktricks. Remember to send a pong back if they send a ping or your connection will time out.

```jsx
#Connection with random nickname
USER ran213eqdw123 0 * ran213eqdw123
NICK ran213eqdw123
#If a PING :<random> is responded you need to send
#PONG :<received random>

VERSION
HELP
INFO
LINKS
HELPOP USERCMDS
HELPOP OPERCMDS
OPERATOR CAPA
ADMIN      #Admin info
USERS      #Current number of users
TIME       #Server's time
STATS a    #Only operators should be able to run this
NAMES      #List channel names and usernames inside of each channel -> Nombre del canal y nombre de las personas que estan dentro
LIST       #List channel names along with channel banner
WHOIS <USERNAME>      #WHOIS a username
USERHOST <USERNAME>   #If available, get hostname of a user
USERIP <USERNAME>     #If available, get ip of a user
JOIN <CHANNEL_NAME>   #Connect to a channel

#Operator creds Brute-Force
OPER <USERNAME> <PASSWORD>
```

I did this

```bash
 > LIST 
:irc.spaghetti.lan 321 adler Channel :Users Name
:irc.spaghetti.lan 322 adler #mailAssistant 1 :[+nt] 
:irc.spaghetti.lan 323 adler :End of channel list.

 > JOIN #mailAssistant
:adler!adler@192.168.49.121 JOIN :#mailAssistant
:irc.spaghetti.lan 353 adler = #mailAssistant :@spaghetti_BoT adler
:irc.spaghetti.lan 366 adler #mailAssistant :End of /NAMES list.

 > WHOIS spaghetti_BoT
:irc.spaghetti.lan 311 adler spaghetti_BoT spaghetti_ 127.0.0.1 * :python
:irc.spaghetti.lan 319 adler spaghetti_BoT :@#mailAssistant
:irc.spaghetti.lan 312 adler spaghetti_BoT irc.spaghetti.lan :Spaghetti IRC Server
:irc.spaghetti.lan 317 adler spaghetti_BoT 844 1675231031 :seconds idle, signon time
:irc.spaghetti.lan 318 adler spaghetti_BoT :End of /WHOIS list.
```

You can send an IRC message in the channel like this:

```bash
privmsg #mailAssistant hello
## or
msg #mailAssistant hello
## or 
query #mailAssistant hello
```

This sends “hello” to the channel `#mailAssistant`.

To send a message to the user `spaghetti_BoT` within the channel (you must have done `JOIN #channel` first)

```bash
privmsg spaghetti_BoT hello
```