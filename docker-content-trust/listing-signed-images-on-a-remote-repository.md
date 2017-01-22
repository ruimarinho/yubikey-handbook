### Listing signed images on a remote repository

Let's list the existing images available on a private registry using a private Notary server:

```sh
❯ notary -s https://127.0.0.1:4443 -d ~/.docker/trust list <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

NAME     DIGEST                                                              SIZE (BYTES)    ROLE
----     ------                                                              ------------    ----
1.0.0    475d897467451caf22f22ad9fd2856a5dd4a876b9eb2daab4d474185f4244e8d    2102            targets
1.0.1    afc214501f950ca11246ceb62fb5c07dd5856f359d6122a620e2c60071a484bc    2537            targets
```

This also works on the official Docker Hub registry by pointing to the public Notary service (you'll have to prepend `docker.io/library`):

```sh
❯ notary -s https://notary.docker.io -d ~/.docker/trust list docker.io/library/alpine
```
