#### gpg failed to sign the data

If you get the following messages when trying to sign a commit or tag:

```
error: gpg failed to sign the data
error: unable to sign the tag
```

First, attempt to remove and re-insert the Yubikey. Then, make sure the card status lists correctly:

```
❯ gpg --card-status
```

If you see:

```
PIN retry counter : 0 0 3
```

This means you have blocked the normal PIN due to many incorrect attempts. The third PIN represents the retry counter for the Admin PIN.

Unblock the normal PIN by entering the Admin PIN:

```
❯ gpg --card-edit

gpg/card> admin
Admin commands are allowed

gpg/card> passwd
gpg: OpenPGP card no. … detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 2
PIN unblocked and new PIN set.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? q
```
