# Brute force attack

If there’s a web server running, use cewl to curate a wordlist first (before trying cupp)

### Cupp

If not cewl, try with Cupp https://github.com/Mebus/cupp which makes a dictionary based on first and last names (if you have a user). This is good for the OSCP. Like this

```
└─$ python3 cupp.py -i
 ___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]

[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: jane
> Surname: foster
................................................................
[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to jane.txt, counting 392 words.
> Hyperspeed Print? (Y/n) : Y
[+] Now load your pistolero with jane.txt and shoot! Good luck!
```

********************************************(This was from a webserver that revealed the name of a user to be jane foster).********************************************

Use the generated wordlist (you might have to wait a minute for it to generate) to do a brute force attack. Use it for both the username and password fields

![Untitled](Brute%20force%20attack%20b4148bfd5fca490a9d9e60f673cc33c1/Untitled.png)

Hydra doesn’t always work, though. It will sometimes produce false negatives. So in order to remedy this, use your wordlist in burp suite Intruder to perform a brute force attack.

The generated wordlist isn’t that big so it shouldn’t take long to brute force in intruder.

The best way is to use a cluster bomb. Make the first payload usernames and the second passwords (in this case because usernames come first in the http request)

![Untitled](Brute%20force%20attack%20b4148bfd5fca490a9d9e60f673cc33c1/Untitled%201.png)

![Untitled](Brute%20force%20attack%20b4148bfd5fca490a9d9e60f673cc33c1/Untitled%202.png)

![Untitled](Brute%20force%20attack%20b4148bfd5fca490a9d9e60f673cc33c1/Untitled%203.png)

In my case, the right creds `admin:Foster2020` gives a different size result than the others.

![Untitled](Brute%20force%20attack%20b4148bfd5fca490a9d9e60f673cc33c1/Untitled%204.png)

## Crunch

Another option for wordlists is crunch. The following command will product the following output

```bash
crunch 24 24 -t ThisIsTheUsersPassword%% -o passwords.txt
## output 
ThisIsTheUsersPassword00
ThisIsTheUsersPassword01
-- snip --
ThisIsTheUsersPassword97
ThisIsTheUsersPassword98
ThisIsTheUsersPassword99
```

Finally, use rockyou.txt with hydra or medusa