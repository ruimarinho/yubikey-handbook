### Touch protection

The Yubikey 4 introduces a new touch feature<sup>1</sup>that enables a second layer of protection when using a private key stored on the device. The access will be conditioned by a user physically triggering the touch sensor, which detracts malware issuing command on the Yubikey without user knowledge.

The touch event is requested for up to 15 seconds, after which the Yubikey turns off the notification.

The touch sensor can be configured with the following parameters:

- **off**: touch is disabled
- **on**: touch is enabled
- **fix**: touch is enabled and can not be disabled unless a new private key is generated or imported.

Touch protection can be configured individually on each one of the GPG private keys and requires the use of the Admin PIN.

**References**

1. <https://developers.yubico.com/PGP/Card_edit.html>
