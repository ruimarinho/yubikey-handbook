#### Signing pushes

`git push` can be instructed to sign the push. The server may use this to control the execution of certain hooks:

```
❯ git push --signed
```

Right now, GitHub does not appear to have support for signed pushes.
