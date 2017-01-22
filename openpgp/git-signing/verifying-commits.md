#### Verifying commits

Import the public GPG key of the signer first and then use of any of the following commands to check for the `Good signature` message:

```sh
❯ git show HEAD --show-signature
❯ git log --show-signature
❯ git verify-commit HEAD
```
