### Generating the Key Revocation List (KRL)

The KRL is a compact binary format which allows revoking SSH signed certificates.

1. Create an empty revoking list:

  ```sh
  ❯ touch /etc/ssh/revoked_keys
  ```

2. Update `/etc/ssh/sshd_config` to include the new Key Revocation List:

  ```sh
  ❯ RevokedKeys /etc/ssh/revoked_keys
  ```

3. When necessary, revoke the first signed certificate:

  ```sh
  ❯ ssh-keygen -k -f revoked_keys -s sshuser.root.ca.pub foo-cert.pub
  ```

4. When necessary, append more revoked certificates (using `-u`):

  ```sh
  ❯ ssh-keygen -k -f revoked_keys -s sshuser.root.ca.pub -u bar-cert.pub
  ```

5. Confirm that revocation worked:

  ```sh
  ❯ ssh-keygen -Qf revoked_keys foo-cert.pub
  ```

6. Distribute the updated `revoked_keys` to every host (`/etc/ssh/revoked_keys`) using `rsync`, `scp` or other orchestration utility.

NOTE: `ssh-keygen` should not require the signed public certificate to revoke it. Instead, using just the serial number should work. However, this is currently not working on OpenSSH 7.2p2 (Ubuntu).
