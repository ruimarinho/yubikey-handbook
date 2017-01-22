#### Automating image signing on CI systems

1. Create a delegation key for the CI system.
2. Expose an encrypted passphrase in the CI environment that imports the key via the local notary client:

  ```sh
  ❯ export NOTARY_DELEGATION_PASSPHRASE=foobar
  ❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust key import ./delegation.key --role user
  ```

3. Use the encrypted passphrase to sign and push the image:

  ```sh
  ❯ export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE=$NOTARY_DELEGATION_PASSPHRASE
  ❯ DOCKER_CONTENT_TRUST_SERVER=https://notary.docker.io docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:latest .
  ❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:latest
  ```
