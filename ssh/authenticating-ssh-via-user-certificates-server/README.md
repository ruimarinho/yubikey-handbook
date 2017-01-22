## Authenticating SSH via User Certificates (server)

A complicated aspect of security is reliability and guaranteeing the consistency of all security controls. Instead of relying on a central authentication authority such as LDAP or Kerberos, we can take advantage of SSH or, more specifically OpenSSH, to provide both.

In addition to [authenticating SSH client access with PIV and PKCS#11](#authenticating-ssh-client-access-with-piv-and-pkcs-11), it is possible to increment the security of the remote SSH authentication. Facebook and Yahoo have switched to SSH User Certificates to avoid lockdown if the central authentication system goes down. It also helps maintaining the `authorized_keys` file, as it does not scale well (it requires a 1:1 match).

A _SSH User Certificate Authority_ can sign and thus securely authenticate each client connecting to a server.

The signed certificate also designates the principals (login identities) that can be used with that certificate. For each user, the principals can be described on a file:

```sh
❯ mkdir /etc/ssh/auth_principals
❯ echo -e 'access-root' > /etc/ssh/auth_principals/root
❯ echo -e 'access-databases' > /etc/ssh/auth_principals/foobar
```

In this example, any signed certificate with the `access-root` principal would be allowed to SSH into that host with the `root` username, and any signed certificate with the `access-databases` principal would be able to login with the `foobar` user.

Now let's create the _SSH User Certificate Authority_.

1. Using an air gapped computer, generate the user certificate authority:

  ```sh
  ❯ ssh-keygen -C "SSH User Certificate Authority" -f sshuser.root.ca
  ```

2. Distribute the public key (`sshuser.root.ca.pub`) to `/etc/ssh/` on every host. Make sure the file is `chmod 644`.

3. Update `/etc/ssh/sshd_config` to include the new CA and principals file:

  ```sh
  TrustedUserCAKeys /etc/ssh/sshuser.root.ca.pub
  AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u
  ```

4. Have the user/client extract the public key from their Yubikey so it can be signed by the new CA on the air gapped computer:

  ```sh
  ssh-keygen -D /usr/local/opt/opensc/lib/pkcs11/opensc-pkcs11.so -e
  ```

5. Sign the user certificate on the air gapped computer, with particular attention to the login name (`<user>`), the principals which this certificate will be able to claim (`<principals>`, separated by commas), the certificate expiration time (`+52w`) and the serial number (`<serial>`, an integer which should be tracked):

  ```sh
  ❯ ssh-keygen -s sshuser.root.ca -I <user> -n <principals> -V +52w -z <serial> <user>.pub

  Signed user key foobar-cert.pub: id "foobar" serial 1928121 for access-root valid from 2016-12-10T00:10:00 to 2017-12-09T00:10:10
  ```

6. Confirm the user certificate looks good:

  ```sh
  ❯ ssh-keygen -Lf <user>-cert.pub

  user-cert.pub:
      Type: ssh-rsa-cert-v01@openssh.com user certificate
      Public key: RSA-CERT SHA256:NWmw3siRlxn3bsIhzaFrCsh66KKIWapFuZsNiDXhRLw
      Signing CA: RSA SHA256:HLD1Eb4XiCoyXew23skyisJt+3P02MOsrHHbK/DmlgY
      Key ID: "foobar"
      Serial: 1928121
      Valid: from 2016-12-10T00:10:00 to 2017-12-09T00:10:10
      Principals:
              access-root
      Critical Options: (none)
      Extensions:
              permit-X11-forwarding
              permit-agent-forwarding
              permit-port-forwarding
              permit-pty
              permit-user-rc
  ```

7. Copy `<user>-cert.pub` to the client's `~/.ssh` directory and name it `id_rsa-cert.pub`. The name is quite specific, as there seems to be a limitation on `opensc-pkcs11` to detect a certificate other than `id_rsa-cert.pub`.
