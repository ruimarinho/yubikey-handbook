### Threshold validation signing

One of the most exciting features that Docker Content Trust will enable in the future is the concept of _threshold validation signing_, which will allow staged verification signing. This will enable verification pipelines such as making sure that an image can only be deployed to staging after being signed by the CI system, or that an image can only be deployed to production once certain subset of keys is present on the image's signature (user key, CI key, staging key and QA key).

There will also be a possibility of defining signing thresholds within a single role (i.e. requiring just one 1 out of 5 CI keys, 2 out of 4 QA keys, etc).

Discussion is actively happening on GitHub:

- [Implement Threshold Signing mechanisms on the client/server](https://github.com/docker/notary/issues/841)
- [TAP3](https://github.com/theupdateframework/taps/blob/master/tap3.md)
- [TAP4](https://github.com/theupdateframework/taps/blob/master/tap4.md)
