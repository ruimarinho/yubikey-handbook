#### Verifying tags

Import the public GPG key of the signer first and then use of any of the following commands to check for the `Good signature` message:

```sh
❯ git tag -v 1.0.0
❯ git cat-file -p 1.0.0
❯ git verify-tag 1.0.0
```
