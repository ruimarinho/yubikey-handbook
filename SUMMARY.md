# Summary

- [Introduction](README.md)
  - [About the author](introduction/about-the-author.md)
- [Personal Identity Verification (PIV)](piv/README.md)
	- [Use cases for a PIV-enabled Yubikey](piv/use-cases.md)
	- [Yubikey PIV Manager](piv/yubikey-piv-manager.md)
- [Device initialization](device-initialization/README.md)
- [Authenticating SSH with PIV and PKCS#11 (client)](ssh/authenticating-ssh-with-piv-and-pkcs11-client/README.md)
	- [Troubleshooting](ssh/authenticating-ssh-with-piv-and-pkcs11-client/troubleshooting.md)
- [Authenticating SSH via User Certificates (server)](ssh/authenticating-ssh-via-user-certificates-server/README.md)
	- [Generating the Key Revocation List (KRL)](ssh/authenticating-ssh-via-user-certificates-server/generating-the-key-revocation-list-krl.md)
- [Authenticating SSH Host Certificates (client)](ssh/authenticating-ssh-host-certificates-client/README.md)
	- [Additional resources](ssh/authenticating-ssh-host-certificates-client/additional-resources.md)
- [2FA via Yubico OTP (server)](yubico-otp/README.md)
	- [Setting up a remote server](yubico-otp/setting-up-a-remote-server/README.md)
		- [Prerequisites (demonstration only)](yubico-otp/setting-up-a-remote-server/prerequisites-demonstration-only.md)
		- [Configuring OpenSSH (sshd) for 2FA authentication](yubico-otp/setting-up-a-remote-server/configuring-openssh-sshd-for-2fa-authentication.md)
		- [Installing libpam-yubico](yubico-otp/setting-up-a-remote-server/installing-libpam-yubico.md)
		- [Creating the Yubikey PAM authentication policy](yubico-otp/setting-up-a-remote-server/creating-the-yubikey-pam-authentication-policy.md)
		- [Yubikey authentication module](yubico-otp/setting-up-a-remote-server/yubikey-authentication-module.md)
	- [Testing](yubico-otp/testing.md)
- [OATH (TOTP and HOTP)](oath/README.md)
	- [Using the Yubico Authenticator](oath/yubico-authenticator.md)
- [U2F (Security Keys)](u2f/README.md)
- [Docker Content Trust](docker-content-trust/README.md)
	- [Key Management](docker-content-trust/key-management.md)
	- [Running Notary services](docker-content-trust/notary/README.md)
		- [Configuring Notary](docker-content-trust/notary/configuring.md)
		- [Managing certificates](docker-content-trust/notary/certificates.md)
		- [Additional resources](docker-content-trust/notary/additional-resources.md)
	- [Pushing a signed Docker image](docker-content-trust/pushing-signed-image/README.md)
		- [Generate the root key on the Yubikey](docker-content-trust/pushing-signed-image/generating-the-root-key.md)
		- [Pushing the image](docker-content-trust/pushing-signed-image/pushing-a-signed-docker-image.md)
	- [Listing signed images on a remote repository](docker-content-trust/listing-signed-images-on-a-remote-repository.md)
	- [Delegation roles](docker-content-trust/delegation-roles/README.md)
		- [Generating a delegation key](docker-content-trust/delegation-roles/generating-a-delegation-key.md)
		- [Importing a delegation certificate](docker-content-trust/delegation-roles/importing-a-delegation-certificate.md)
		- [Using a delegation key](docker-content-trust/delegation-roles/using-a-delegation-key.md)
		- [Automating image signing on CI systems](docker-content-trust/delegation-roles/automating-image-signing-on-ci-systems.md)
	- [Removing a delegation key](docker-content-trust/removing-a-delegation-key.md)
	- [Rotating a key](docker-content-trust/key-rotation/README.md)
		- [Snapshot key](docker-content-trust/key-rotation/snapshot-key.md)
		- [Timestamp key](docker-content-trust/key-rotation/timestamp-key.md)
		- [Targets key](docker-content-trust/key-rotation/targets-key.md)
	- [Threshold validation signing](docker-content-trust/threshold-validation-signing.md)
- [OpenPGP](openpgp/README.md)
	- [Touch protection](openpgp/touch-protection/README.md)
		- [Enabling touch protection](openpgp/touch-protection/enabling-touch-protection.md)
	- [Importing keys](openpgp/importing-keys.md)
	- [Editing metadata](openpgp/editing-metadata.md)
	- [Git signing](openpgp/git-signing/README.md)
		- [Signing tags](openpgp/git-signing/signing-tags.md)
		- [Verifying tags](openpgp/git-signing/verifying-tags.md)
		- [Signing commits](openpgp/git-signing/signing-commits.md)
		- [Verifying commits](openpgp/git-signing/verifying-commits.md)
		- [Signing merges](openpgp/git-signing/signing-merges.md)
		- [Signing pushes](openpgp/git-signing/signing-pushes.md)
	- [Authenticating SSH with GPG](openpgp/authenticating-ssh-with-gpg.md)
	- [Troubleshooting](openpgp/troubleshooting/README.md)
		- [gpg failed to sign the data](openpgp/troubleshooting/gpg-failed-to-sign-the-data.md)
- [macOS integration](macos/README.md)
	- [Offline authentication using HMAC-SHA1 Challenge-Response](macos/offline-authentication/README.md)
		- [Configuring HMAC-SHA1 Challenge-Response](macos/offline-authentication/configuration.md)
	- [Login and keychain authentication](macos/login/README.md)
		- [Managing pairing](macos/login/managing-pairing.md)