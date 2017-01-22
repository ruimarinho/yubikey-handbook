### Using the Yubico Authenticator

Download [Yubico Authenticator](https://developers.yubico.com/yubioath-desktop/Releases/), then `File > Add`.

Click `Scan a QR code` if a QR code is visible on the screen or manually enter the details. The QR code is usually a visual representation of the following address URL-encoded:

```
otpauth://totp/<email>?issuer=<issuer>&secret=<secret>
```

- `Credential name`: the name of the provider (e.g. `GitHub`)
- `Secret key (base32)`: the secret key (32 characters)
- `Credential type`: usually TOTP, but can be HOTP
- `Number of digits`: usually 6, but can be up to 8
- `Algorithm`: usually SHA-1, but can be SHA-256
- `Require touch`: enable

It's possible to convert OTP secrets stored in 1Password by editing the item's OTP field and parsing the URL as demonstrated above.
