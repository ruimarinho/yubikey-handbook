#### Managing certificates

You should generate your own certificates before going to production.

Depending on the go version used, the `notary-server` certificate [may have](https://go-review.googlesource.com/#/c/10806/) to be marked for both EKUs `clientAuth` (for connection as a gRPC client) _and_ `serverAuth` (for serving TLS as a server).
