### Authenticating SSH with GPG

While technically it's certainly possible to authenticate SSH sessions using GPG, `gpg-agent` does not always have a friendly co-existence with `ssh-agent`.

Using PIV for authenticating SSH remains the recommended solution.

Enable `enable-ssh-support` and `write-env-file` under `~/.gnupg/gpg-agent.conf`:

```
default-cache-ttl 600
  max-cache-ttl 7200
  pinentry-program /usr/local/MacGPG2/libexec/pinentry-mac.app/Contents/MacOS/pinentry-mac
  enable-ssh-support
  write-env-file
```
