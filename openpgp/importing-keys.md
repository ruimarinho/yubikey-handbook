### Importing keys

GPG private keys can be copied or moved to the Yubikey. The following examples moves them.

```
❯ gpg --edit-key <keyId>

gpg> toggle
gpg> key 1
gpg> keytocard

Please select where to store the key:
   (1) Signature key
   (3) Authentication key
Your selection? 1

gpg> key 1
gpg> key 2
gpg> keytocard

Please select where to store the key:
   (2) Encryption key
Your selection? 2

gpg> key 2
gpg> key 3
gpg> keytocard

Please select where to store the key:
   (3) Authentication key
Your selection? 3

gpg> quit
```

Confirm the private keys have been moved to the Yubikey:

```sh
❯ gpg --list-secret-keys
```

If you see `ssb>`, it indicates a stub to the private key on Yubikey, which in this case means the move was successful.

Then, check the card status:

```sh
❯ gpg --card-status
```
