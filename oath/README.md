## OATH (TOTP and HOTP)

The Initiative for Open Authentication (OATH) is responsible for developing two standards - TOTP (clock-based) and HOTP (counter-based). Both are used [extensively](https://twofactorauth.org/) nowadays.

The Yubikey can emit an HOTP token when touched. For TOTP, a companion application ([Yubico Authenticator](https://developers.yubico.com/yubioath-desktop)) must be used as Yubikeys do not have an internal clock.

**Why use a Yubikey for OATH**?

- The shared secrets are stored securely in the Yubikey.
- Can be used on any computer and thus is not conditioned by typical mobile device issues, such as drained battery.
