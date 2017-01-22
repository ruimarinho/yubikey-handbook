#### Configuring Notary

Depending on the environment and purpose of running Notary services, there are two options: using `docker-compose` when running locally or running each service separately, usually through an orchestration layer (Kubernetes, Rancher, Swarm and so on). Configuring the latter is outside the scope of this document, while the former should only be used for demonstration purposes.

The following examples assume you will run Notary on your internal infrastructure and store Docker images on Amazon ECR.

##### Using docker-compose

The following examples assume that `postgres` was the choice of your `backend`.

Start by cloning the official repository:

```sh
❯ git clone https://github.com/docker/notary.git
❯ git checkout v0.5.0
```

You can define the configuration via environment variables on the `docker-compose.postgres.yml` file that will override config settings. Alternatively you can also edit `fixtures/server-config.postgres.json` and `fixtures/signer-config.postgres.json`. If you choose to go with the default, no change is required.

Note that if you choose to use `fixtures/regenerateTestingCerts.sh` for local testing with `Docker for Mac.app` using Docker-in-Docker for simulating multiple clients, make sure to add `IP:192.168.65.1` to the `subjectAltName` or `192.168.65.1 notaryserver` to `/etc/hosts`, otherwise any Docker operation will result in a invalid certificate error.

If you have bootstrapped Notary before, you may need to delete its volume to get the new certificates reloaded (`docker volume rm notary_notary_data`). Beware that this will destroy all previously stored testing data.

```sh
docker-compose -f docker-compose.postgresql.yml up
```

If you chose not to sign the certificates using a previously-trusted internal CA, you will need to manually trust the `fixtures/root-ca.crt`, otherwise Docker will throw out this error when interacting with Notary:

```sh
ERRO[0000] could not reach https://127.0.0.1:4443: Get https://127.0.0.1:4443/v2/: x509: certificate signed by unknown authority
```

Trust the self-signed Notary's Intermediate CA:

```sh
❯ mkdir -p ~/.docker/tls/127.0.0.1:4443/
❯ cp fixtures/intermediate-ca ~/.docker/tls/127.0.0.1:4443/
```

From now on, Docker can be instructed to always use the private Notary server:

```sh
❯ export DOCKER_CONTENT_TRUST=1
❯ export DOCKER_CONTENT_TRUST_SERVER=https://127.0.0.1:4443
```

A completely separate environment can be created by using Docker-in-Docker, where delegations can be explored:

```sh
❯ docker run -d --privileged -e DOCKER_CONTENT_TRUST=1 -e DOCKER_CONTENT_TRUST_SERVER=https://notaryserver:4443 docker:dind
905c2a005f34745175299da9652a991860f965bbd1e24642e20c6abb9de03174
```

Then enter the container:

```sh
❯ docker exec -it 905c2a005f34745175299da9652a991860f965bbd1e24642e20c6abb9de03174 sh

/ # echo "192.168.65.1 notaryserver" >> /etc/hosts
/ # wget -O /usr/local/bin/notary https://github.com/docker/notary/releases/download/v0.4.2/notary-Linux-amd64 && chmod a+x /usr/local/bin/notary
```

Make sure to add the Notary server certificate to `/root/.docker/tls/notaryserver:4443/notaryserver.crt`.

If using Amazon ECR for a private registry and credentials are needed, echo them to a local file and pipe it to shell:

```
❯ cat aws.login | sh
```
