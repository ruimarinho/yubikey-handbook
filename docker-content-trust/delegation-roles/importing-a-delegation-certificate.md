#### Importing a delegation certificate

As a repository owner:

1. Add the delegation key to the repository using the `targets/releases` path, as this is what Docker searches for when signing an image (first `targets`, then `targets/releases`:

  ```sh
  ❯ notary delegation -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust add <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app targets/releases delegation.crt --all-paths

  Addition of delegation role targets/releases with keys [e93a68479026f002b9dedb35f563cf5abc50aecd18b2205ad296d3101c0d3c21], with paths ["" <all paths>], to repository "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app" staged for next publish.
  ```

2. Check the unpublished (staged) changes:

  ```sh
  ❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust status <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

  Unpublished changes for <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:

  #    ACTION    SCOPE               TYPE          PATH
  -    ------    -----               ----          ----
  0    create    targets/releases    delegation
  1    create    targets/releases    delegation
  ```

3. Publish the queued changes:

  ```sh
  ❯ notary -v -s https://127.0.0.1:4443 -d ~/.docker/trust publish <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

  Pushing changes to <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app
  Enter passphrase for targets key with ID 79a6fca:
  Successfully published changes for repository <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app
  ```

  Enter the previously created `targets` repository key passphrase.

4. Confirm the delegation list looks correct:

  ```
  ❯ notary -v -s https://127.0.0.1:4443 -d ~/.docker/trust delegation list <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

  ROLE                PATHS             KEY IDS                    THRESHOLD
  ----                -----             -------                    ---------
  targets/releases    "" <all paths>    61a6430…                   1
  ```
