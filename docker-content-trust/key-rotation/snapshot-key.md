#### Snapshot key

To rotate the `snapshot` key:

```sh
❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust key rotate <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app snapshot -r

Enter passphrase for new targets key with ID 3c10139 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Repeat passphrase for new targets key with ID 3c10139 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Enter the User Pin for the attached Yubikey:
Please touch the attached Yubikey to perform signing.

Successfully rotated targets key for repository <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app
```
