### Editing metadata

Now that the private keys have been moved to the Yubikey, it's time to configure some basic properties, such as changing the default management PIN (what GPG calls the _Admin PIN_) and also the regular PIN whenever the private key is accessed.

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

Your selection? 3
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
PIN changed.

gpg/card> q

gpg/card> login
foobar

gpg/card> sex
m

gpg/card> name
Cardholder\'s surname: Bar
Cardholder\'s given name: Foo

gpg/card> lang
Language preferences: en

gpg/card> login
Login data (account name): foobar

gpg/card> list

gpg/card> quit
```
