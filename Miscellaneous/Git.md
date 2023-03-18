# Git

# Git Shell

If you have an interactive git shell, you can clone a remote git repo onto your computer for privilege escalation.

```abap
# clone repo
GIT_SSH_COMMAND='ssh -i id_rsa -p 43022' git clone git@192.168.116.125:/git-server
# commit a change
GIT_SSH_COMMAND='ssh -i id_rsa -p 43022' git commit backups.sh
# push changes
GIT_SSH_COMMAND='ssh -i ../id_rsa -p 43022' git push origin master
```

![Untitled](Git%2004aaaca56faa4d56827c436777c9957b/Untitled.png)

![Untitled](Git%2004aaaca56faa4d56827c436777c9957b/Untitled%201.png)