### Key Management

Content trust is directly associated with an image tag and each repository has a set of keys that publishers use to sign each image.

A repository can have both unsigned and signed images. They live as separate entities, so the same tag (e.g. `latest`) can point to different contents depending on whether Docker Content Trust is enabled or not on the client.

Image trust builds on 4 keys:

- A `root` key (_offline_) which is the root anchor of the content trust for an image. This is key that gets stored on the Yubikey and only brought online for a limited number of operations
- A `targets` key (_online_) - the key that signs the actual files downloaded, stored on the client and encrypted at rest
- A `snapshot` key (_online_), which signs the metadata file containing information about all the other metadata available on the collection
- A `timestamp` key (_online_), which ensures content freshness by periodically signing a timestamped statement.

The `snapshot` and the `timestamp` can be managed by the Notary service for convenience.
