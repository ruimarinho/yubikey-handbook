## Authenticating SSH with PIV and PKCS#11 (client)

One of the coolest features of the Yubikey is authenticating SSH sessions via PKCS#11. The private key is stored on the Yubikey and whenever it is accessed, Yubikey can require a touch action.

Besides the common remote login, all connections that use SSH, such as remote git server (e.g. GitHub), may trigger this behavior if desired. This is a protection on the client side to prevent unauthorized SSH private key access.

Note that only RSA keys are supported when using this method.

1. Create a 2048-bit RSA key pair:

  ```sh
  ❯ yubico-piv-tool -s 9a -a generate -k --pin-policy=once --touch-policy=always --algorithm=RSA2048 -o public.pem
  ```

  Enter Yubikey's _Management key_.

2. Create a self-signed certificate (or, alternatively, a certificate signing request):

  ```sh
  ❯ yubico-piv-tool -a verify-pin -a selfsign-certificate -s 9a -S '/CN=ssh/' --valid-days=365 -i public.pem -o cert.pem
  ```

  Enter PIN then touch the Yubikey.

  Alternatively replace `selfsign-certificate` by `request-certificate` and send the resulting `.csr` file for internal CA certification.

3. Import the (self-)signed certificate:

  ```sh
  ❯ yubico-piv-tool -k -a import-certificate -s 9a -i cert.pem
  ```

  Enter Yubikey's _Management key_.

4. Export the public key stored in the Yubikey in the correct format for OpenSSH:

  ```sh
  ❯ ssh-keygen -D /usr/local/opt/opensc/lib/pkcs11/opensc-pkcs11.so -e
  ```

  Alternatively, you can use the generated public key before you delete it:

  ```sh
  ❯ ssh-keygen -i -m PKCS8 -f public.pem
  ```

  You can now share this public key for SSH authentication (e.g `~/.ssh/authorized_keys`).

5. Check slot 9a status (optional):

  ```sh
  ❯ yubico-piv-tool -a status
  ```

6. Add the SSH key provided via PKCS#11 to the local ssh-agent:

  ```sh
  ❯ ssh-add -s /usr/local/opt/opensc/lib/pkcs11/opensc-pkcs11.so
  ```

  Enter the Yubikey PIN when it asks for the passphrase.

7. Confirm the key has been added:

  ```sh
  ❯ ssh-add -L
  ```

8. Alternatively, selectively add the PKCS11Provider to `~/.ssh/config`:

  ```sh
  Host github.com
    PKCS11Provider /usr/local/opt/opensc/lib/pkcs11/opensc-pkcs11.so
    Port 22
    User foobar
  ```
