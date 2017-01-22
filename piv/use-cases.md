### Use cases for a PIV-enabled Yubikey

Yubikey 4 comes with 5 programmable slots capable of holding X.509 certificates, together with it's accompanying private key. The other 20 remaining slots are used for _Retired Key Management_, which allows decryption of earlier encrypted documents or emails using retired keys.

Each slot has a predestined use case, including a default PIN and touch policy. However, these are merely indicative and may be overwritten.

**Slot 9a: PIV Authentication**

Authenticates the smart card and the cardholder (e.g. OS logins, ssh, WiFi, OpenVPN, curl, Android code, Mac code, automatic screen locking). By default, PIN is required once and may be re-used for subsequent operations.

**Slot 9c: Digital Signature**

Signs objects (Android codesign, Mac codesign, storing intermediate CA private key). By default, PIN is required for each signing operation.

**Slot 9d: Key Management**

Encrypts object for the purpose of confidentiality. By default, PIN is required once and may be re-used for subsequent operations.

**Slot 9e: Card Authentication**

Authenticates against physical access applications (e.g. door locks). By default, PIN is not required.

**Slot f9: Attestation**

Attests other slot keys were generated on the device.
