#### Generating a delegation key

Make sure you're using OpenSSL 1.0.2+, otherwise the keys will be signed with SHA1, which Notary will refuse to import (_invalid SHA1 signature algorithm_):

```sh
❯ brew install openssl
```

Make sure the `openssl` in your `$PATH` points to the correct version:

```sh
❯ openssl version
OpenSSL 1.0.2j  26 Sep 2016
```

Have the collaborator generate a new key pair on their machine (you can use the Docker-in-Docker container for this):

1. Create an 256-bit ECC key pair:

  ```sh
  ❯ openssl ecparam -name prime256v1 -genkey -out delegation.key -noout
  ```

2. Create a certificate signing request:

  ```sh
  ❯ openssl req -new -sha256 -key delegation.key -out delegation.csr
  ```

3. Self-sign the certificate (or, alternatively, sign the csr using an internal CA):

  ```sh
  ❯ openssl x509 -req -days 365 -in delegation.csr -signkey delegation.key -out delegation.crt
  ```

4. Import the private key:

  ```
  ❯ notary -v -s https://127.0.0.1:4443 -d ~/.docker/trust key import delegation.key --role user --gun <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

  Enter passphrase for new user key with ID e93a684 (tuf_keys):
  Repeat passphrase for new user key with ID e93a684 (tuf_keys):
  ```

  Enter a passphrase generated and stored on a password manager.

5. Send the public key (`delegation.crt`) to the repository owner.
