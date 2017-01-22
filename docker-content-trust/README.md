## Docker Content Trust

Docker Content Trust allows delivering trusted images over in insecure network using signed containers. Docker Engine can verify the integrity and freshness of an image along the entire Docker flow (`push`, `pull`, `build`, `create` and `run` operations). It is commonly referred as an opinionated integration of Notary in Docker.

Notary is a tool based on [The Update Framework](http://theupdateframework.com/) that solves the problem of secure software updates delivery over the network. It provides mitigations for the following known attacks:

- Image tampering, by digitally signing each layer
- Key compromise, by providing transparent key rotation mechanisms out of the box
- Replay attacks, by using a timestamped key that ensures content freshness

Docker Hub runs a Notary service against which official signed images can be verified. When running a private registry (e.g. Amazon ECR), a private Notary service is required.
