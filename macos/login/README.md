### Login and keychain authentication

macOS Sierra (10.12) ships with an improved SmartCard integration, including support for SmartCard Extensions<sup>1</sup>, an updated CCID driver and bug fixes on the PC/SC framework<sup>2</sup>. This translates into a clean and simple configuration for using a Yubikey to login to the macOS.

The login screen as well as Keychain-related operations will show a _PIN_ placeholder instead of _Password_ if the Yubikey is plugged in and has been paired before.

**References**

1. <https://developer.apple.com/library/content/releasenotes/MacOSX/WhatsNewInOSX/Articles/OSXv10.html>

2. <https://ludovicrousseau.blogspot.pt/2016/09/macos-sierra-and-smart-cards-status.html>
