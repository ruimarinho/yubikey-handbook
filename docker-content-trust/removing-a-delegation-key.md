### Removing a delegation key

As a repository owner, remove a key from all delegation roles:

```sh
❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust delegation purge  <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app --key 900b53cd7116e0eda33905ad9f446b93a4620b6762597bebb5ec529ec842b611
```

Or, optionally, you can remove from a single role:

```sh
❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust delegation remove  <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app targets/releases 900b53cd7116e0eda33905ad9f446b93a4620b6762597bebb5ec529ec842b611
```
