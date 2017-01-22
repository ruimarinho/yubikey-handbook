### Testing

If you've enabled `root` login via ssh, you should be able to login in the Docker container using the published `2222` ssh port:

```sh
❯ ssh root@127.0.0.1 -p 2222
Authenticated with partial success.
root@127.0.0.1 password:
```

You should see the _Authenticated with partial success_ text, which means that the authentication against the public key succeeded.

Now, you must enter the user's password and, **without** hitting enter. Long-touch the Yubikey until a newline is entered automatically.

If you consider the password `foobar` for the `root` user, the actual password that will get sent is:

```
root@127.0.0.1 password: foobarcccccccgklgcvnkcvnnegrnhgrjkhlkfhdkclfncvlgj
```

`libpam-yubico` will remove the characters pertaining to the OTP, send it to YubiCloud, and upon success forward the remaining characters to the next PAM module (in this case, `pam_unix.so`) validate the user password.

After 2-3s, you should be logged in! Now, exit and login with `foobar`. Attempt to escalate privileges by doing `su root` and you will see that the Yubikey for the `root` user will be required (the same principle applies - first enter the password followed by the long-touch on the Yubikey).

As you may have noticed that during SSH, there are actually three factors involved, not two - public key authentication, password and Yubikey OTP. This is actually a limitation of OpenSSH, as public key authentication plus Yubikey OTP without requiring the user's UNIX password is not possible at the moment.
