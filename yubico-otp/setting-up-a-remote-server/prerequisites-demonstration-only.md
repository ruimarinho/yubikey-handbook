#### Prerequisites (demonstration only)

You will need a Linux-based box like Ubuntu running OpenSSH:

```sh
❯ docker run --name yubico-ubuntu -p 2222:22 -it ubuntu
```

Inside the Docker container:

```sh
❯ apt-get update
❯ apt-get install -y openssh-server vim
❯ mkdir -p /root/.ssh
```

Add a user for testing purposes:

```sh
❯ adduser foobar
❯ usermod -G sudo foobar
❯ mkdir -p /home/foobar/.ssh
```

Add your public ssh key to both users. First, locally:

```sh
❯ cat ~/.ssh/id_rsa.pub | pbcopy
```

Then:

```sh
❯ echo "<pubkey>" >> /home/foobar/.ssh/authorized_keys
❯ echo "<pubkey>" >> /root/.ssh/authorized_keys
```

If you'd like to ssh with the `root` user, remember to add a password as that will be required later on:

```sh
❯ passwd root
```
