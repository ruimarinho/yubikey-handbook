#### Generating the root key on the Yubikey

1. List all the keys known on the host:

  ```sh
  ❯ notary -d ~/.docker/trust key list
  ```

  You probably don't have any keys available yet. Let's generate one.

2. Create an 256-bit ECC key pair:

  ```sh
  ❯ yubico-piv-tool -s 9c -a generate -k --pin-policy=always --touch-policy=always --algorithm=ECCP256 -o public.pem
  ```

  Enter Yubikeys _Management key_.

3. Create a self-signed certificate (or, alternatively, a certificate signing request):

  ```sh
  ❯ yubico-piv-tool -a verify-pin -a selfsign-certificate -s 9c -S '/CN=root/' --valid-days=365 -i public.pem -o cert.pem
  ```

  Enter PIN then touch the Yubikey.

  Alternatively replace `selfsign-certificate` by `request-certificate` and send the resulting `.csr` file for internal CA certification.

  The `CN=root` is what allows Notary to find the key on the Yubikey, so it should be kept that way.

4. Import the (self-)signed certificate:

  ```sh
  ❯ yubico-piv-tool -k -a import-certificate -s 9c -i cert.pem
  ```

  Enter Yubikey's _Management key_.

You can choose to generate the private key outside the Yubikey, in case you prefer to have a local backup copy. `notary key generate` will generate a private key locally and then find an empty slot to import it on the Yubikey.

When Notary asks for the _SO PIN_, enter the Yubikey's _Management Key_.

You should now have the `root` key available:

```sh
❯ notary -d ~/.docker/trust key list

ROLE    GUN    KEY ID                                               LOCATION
----    ---    ------                                               --------
root           bf98cc496cb05fd2b88b01d3200900ff05ec83a1f3690690f…   Yubikey
```
