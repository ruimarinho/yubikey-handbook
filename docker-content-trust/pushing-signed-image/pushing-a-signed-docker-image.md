#### Pushing the image

Now that a `root` key is available, it's time to initialize the repository on the first push.

Consider this as your app:

```docker
FROM alpine

RUN true
```

Make sure you have all trusted metadata using the official Notary server when building the image by temporarily redefining the content trust server:

```sh
❯ DOCKER_CONTENT_TRUST_SERVER=https://notary.docker.io docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.0 .
```

Then push it upstream, forcing Docker to initialize the signed repository:

```
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.0

The push refers to a repository [<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Pushed
1.0.0: digest: sha256:475d897467451caf22f22ad9fd2856a5dd4a876b9eb2daab4d474185f4244e8d size: 2101
Signing and pushing trust metadata
Enter the User Pin for the attached Yubikey:
Please touch the attached Yubikey to perform signing.
Enter passphrase for new repository key with ID 9c738a6 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Repeat passphrase for new repository key with ID 9c738a6 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Enter the User Pin for the attached Yubikey:
Please touch the attached Yubikey to perform signing.
Finished initializing "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app"
Successfully signed "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.0
```

The repository key passphrase should be generated and stored by a password manager. This will be a personal key, i.e., not intended to be shared. It will be stored, in its encrypted form, under `~/.docker/trust`. The repository key is will be generated with the `targets` role.

```
❯ notary -d ~/.docker/trust key list
```

Now edit the file and generate a new build:

```docker
FROM alpine

RUN true
RUN uname
```

Build it:

```sh
❯ docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.1 .
```

And push it:

```sh
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.1

The push refers to a repository [<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Layer already exists
1.0.1: digest: sha256:afc214501f950ca11246ceb62fb5c07dd5856f359d6122a620e2c60071a484bc size: 2531
Signing and pushing trust metadata
Enter passphrase for repository key with ID 9c738a6 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Successfully signed "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.1
```

On the second push, only the passphrase of the repository key was required.

That's it. Signed builds using an offline root key (Yubikey) and one online repository key.
