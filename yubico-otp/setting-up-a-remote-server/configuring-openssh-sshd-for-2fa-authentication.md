#### Configuring OpenSSH (sshd) for 2FA authentication

Enable strong 2FA authentication by updating the `/etc/ssh/sshd_config` with the following changes:

```sh
# Require public key *and* password authentication. Without this, a valid public
# key would bypass the Yubikey requirement.
AuthenticationMethods publickey,password

# Enable the password authentication backend.
PasswordAuthentication yes

# Disable the keyboard-interactive mode which could be used to ask for the
# password.
ChallengeResponseAuthentication no

# Enable PAM integration for authentication as this is the system that Yubikey
# integrates with.
UsePAM yes
```

If you want to login with the _root_ user via ssh, add or update the `PermitRootLogin` under the same file, replace `prohibit-password` by `yes`:

```sh
# Enable root login via ssh.
PermitRootLogin yes
```

Restart `ssh`. Note that if you're already inside an ssh session, you won't be disconnected.

```sh
❯ service ssh restart
```
