### Running Notary services

In an organization, a internal Notary HA deployment is expected to be available in order for Docker Content Trust to be usable. Hence, this topic assumes familiarity with the [Notary service architecture](https://github.com/docker/notary/blob/master/docs/service_architecture.md).

Notary ships with support for multiple storage backends. Choose one appropriate to your infrastructure and team.

Both `notary-server` and `notary-signer` should be behind a load balancer. The public facing entry is `notary-server` and only serves public data. The `notary-signer` stores the timestamp keys online - it can be compared to an HSM.

The content publisher owns the client side key and is the one responsible for publishing new content.
