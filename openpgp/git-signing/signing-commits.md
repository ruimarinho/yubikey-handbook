#### Signing commits

Continuing on the previous git repository, add a new file and commit it using the `-S` flag (not `-s` as that means `Signed-Off` on the commit command):

```
❯ touch biz
❯ git add biz
❯ git commit -S -m "Add biz"
```

You may enable auto-signing commits by adding the following to `~/.gitconfig.local`:

```
[commit]
    gpgSign = true
```
