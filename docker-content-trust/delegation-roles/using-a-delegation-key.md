#### Using a delegation key

The collaborator can now push to the repository using Docker Content Trust. Docker will automatically choose and pick the right key for the `targets/release` role.

Edit the file on the Docker-in-Docker container:

```docker
FROM alpine

RUN true
RUN uname
RUN echo collaborating
```

Build the new image:

```sh
❯ DOCKER_CONTENT_TRUST_SERVER=https://notary.docker.io docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3 .
```

Push the new image:

```sh
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3

The push refers to a repository [<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Pushed
1.0.3: digest: sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268 size: 2980
Signing and pushing trust metadata
Enter passphrase for repository key with ID e93a684 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Successfully signed "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.3
```

Test on the repository owner side that the image signed by the collaborator is valid:

```sh
❯ docker pull <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3
Pull (1 of 1): <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3@sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268
sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268: Pulling from app
3690ec4760f9: Already exists
Digest: sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268
Status: Downloaded newer image for <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app@sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268
Tagging <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app@sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268 as <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3
```

Notice that the digest from the collaborator matches the one received on the owner side.

Now attempt to edit the Dockerfile on the owner side again:

```docker
FROM alpine

RUN true
RUN uname
RUN date
```

And build it:

```sh
❯ docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.4 .
```

Everything looks good. Now try to push it:

```sh
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.4

The push refers to a repository [<aws_account_id>.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Pushed
1.0.4: digest: sha256:19cbb30c36b9855aff3ccf7b052bbf6032b7acf4510ea311e82a2e51d926fd8d size: 2966
Signing and pushing trust metadata
Failed to sign "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.4 - no valid signing keys for delegation roles
no valid signing keys for delegation roles
```

What happened here was that when delegation was enabled for this repository, Docker now requires keys to be valid under the `targets/releases` role. Remember that the original key, created upon repository initialization (first push), was listed with the `targets` role instead.

So in order to enable the repository owner to also be able to sign images, the owner needs to follow the exact same steps as all collaborators, i.e., creating and adding its owner `targets/release` key to the repository.

While following the collaborator instructions, you may get this error if you have your Yubikey plugged in when running `notary key import`:

```
ERRO[0007] failed to import key to store: yubikey only supports storing root keys, got user for key: 6965a1ee8ff68a211d769243c0b171f90cb03a337d2337cc91650b843a5bc1ff
```

When the import command is ran, Notary assumes that if a Yubikey is plugged in, it should copy the private key there too. However, a Yubikey should only be used for `root` keys, so when attempting to import a `user` key, it throws out this harmless error. In the future, it will likely be ignored.

After you've imported the key, the resulting list should be:

```sh
❯ notary -d ~/.docker/trust key list

ROLE       GUN                          KEY ID                                                              LOCATION
----       ---                          ------                                                              --------
root                                    bf98cc496cb05fd2b88b01d3200900ff05ec83a1f3690690f2c9341976b64728    yubikey
user                                    a726c2f62f2239055b7a1881c12d0de636b62e0a2c1ef21044083c51962f1959    ~/.docker/trust/private
targets    ...st-1.amazonaws.com/app    9c738a648878fab6124f70f78879dc1da89bae6ac0574c0ea6dfa6f20e80816c    ~/.docker/trust/private
```

Continue with the delegation key steps, adding the new to the delegation and publishing the changes. You will be asked to enter your original `targets` key, just like when adding the first collaborator key.

After you're done with your own delegation key, re-issue the push command:

```sh
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.4
The push refers to a repository [<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Layer already exists
1.0.4: digest: sha256:19cbb30c36b9855aff3ccf7b052bbf6032b7acf4510ea311e82a2e51d926fd8d size: 2966
Signing and pushing trust metadata
Enter passphrase for user key with ID a726c2f:
Successfully signed "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.4
```

Notice that the passphrase is for your own delegation key now and the push finally works.
