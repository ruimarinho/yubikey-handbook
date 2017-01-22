### Offline authentication using HMAC-SHA1 Challenge-Response

The new Yubikeys support a new authentication mechanism called HMAC-SHA1 Challenge-Response which enables offline authentication. This is very useful for working machines as internet access is not required for authentication.

The main use case is to require 2FA for escalating privileges to `root`. For example, the Yubikey can be configured to be required in addition to providing the root password.
