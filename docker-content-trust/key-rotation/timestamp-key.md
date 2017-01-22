#### Timestamp key

The `timestamp` key can also be rotated:

```sh
❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust key rotate <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app timestamp -r

Enter the User Pin for the attached Yubikey:
Please touch the attached Yubikey to perform signing.

Successfully rotated timestamp key for repository <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app
```

This will require removing the `timestamp.json` file on each collaborator machine:

```
rm ~/.docker/trust/tuf/<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app/metadata/timestamp.json
```
