# Yubikey Handbook

<!-- TOC depthFrom:2 depthTo:4 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Introduction](#introduction)
- [Personal Identity Verification (PIV)](#personal-identity-verification-piv)
	- [Use cases for a PIV-enabled Yubikey](#use-cases-for-a-piv-enabled-yubikey)
	- [Yubikey PIV Manager](#yubikey-piv-manager)
- [Device initialization](#device-initialization)
- [Authenticating SSH with PIV and PKCS#11 (client)](#authenticating-ssh-with-piv-and-pkcs11-client)
	- [Troubleshooting](#troubleshooting)
- [Authenticating SSH via User Certificates (server)](#authenticating-ssh-via-user-certificates-server)
	- [Generating the Key Revocation List (KRL)](#generating-the-key-revocation-list-krl)
- [Authenticating SSH Host Certificates (client)](#authenticating-ssh-host-certificates-client)
	- [Additional resources](#additional-resources)
- [2FA via Yubico OTP (server)](#2fa-via-yubico-otp-server)
	- [Setting up a remote server](#setting-up-a-remote-server)
		- [Prerequisites (demonstration only)](#prerequisites-demonstration-only)
		- [Configure OpenSSH (sshd) for 2FA authentication](#configure-openssh-sshd-for-2fa-authentication)
		- [Install libpam-yubico](#install-libpam-yubico)
		- [Create the Yubikey PAM authentication policy](#create-the-yubikey-pam-authentication-policy)
		- [Yubikey authentication module](#yubikey-authentication-module)
	- [Testing](#testing)
- [OATH (TOTP and HOTP)](#oath-totp-and-hotp)
	- [Using the Yubico Authenticator](#using-the-yubico-authenticator)
- [U2F (Security Keys)](#u2f-security-keys)
- [Docker Content Trust](#docker-content-trust)
	- [Key management](#key-management)
	- [Running Notary services](#running-notary-services)
		- [Configuring Notary](#configuring-notary)
		- [Managing certificates](#managing-certificates)
		- [Additional resources](#additional-resources)
	- [Pushing a signed Docker image](#pushing-a-signed-docker-image)
		- [Generate the root key on the Yubikey](#generate-the-root-key-on-the-yubikey)
		- [Pushing the image](#pushing-the-image)
	- [Listing signed images on a remote repository](#listing-signed-images-on-a-remote-repository)
	- [Delegation roles](#delegation-roles)
		- [Generating a delegation key](#generating-a-delegation-key)
		- [Importing a delegation certificate](#importing-a-delegation-certificate)
		- [Using a delegation key](#using-a-delegation-key)
		- [Automating image signing on CI systems](#automating-image-signing-on-ci-systems)
	- [Removing a delegation key](#removing-a-delegation-key)
	- [Rotating a key](#rotating-a-key)
		- [Snapshot key](#snapshot-key)
		- [Timestamp key](#timestamp-key)
		- [Targets key](#targets-key)
	- [Threshold validation signing](#threshold-validation-signing)
- [OpenPGP](#openpgp)
	- [Touch protection](#touch-protection)
		- [Enabling touch protection](#enabling-touch-protection)
	- [Importing keys](#importing-keys)
	- [Editing metadata](#editing-metadata)
	- [Git signing](#git-signing)
		- [Signing tags](#signing-tags)
		- [Verifying tags](#verifying-tags)
		- [Signing commits](#signing-commits)
		- [Verifying commits](#verifying-commits)
		- [Signing merges](#signing-merges)
		- [Signing pushes](#signing-pushes)
	- [Authenticating SSH with GPG](#authenticating-ssh-with-gpg)
	- [Troubleshooting](#troubleshooting)
		- [gpg failed to sign the data](#gpg-failed-to-sign-the-data)
- [macOS integration](#macos-integration)
	- [Offline authentication using HMAC-SHA1 Challenge-Response](#offline-authentication-using-hmac-sha1-challenge-response)
		- [Configuring HMAC-SHA1 Challenge-Response](#configuring-hmac-sha1-challenge-response)
	- [Login and keychain authentication](#login-and-keychain-authentication)
		- [Managing pairing](#managing-pairing)

<!-- /TOC -->

## Introduction

Yubikey is an hardware device manufactured by Yubico that provides several forms of strong authentication and encryption. It has many use cases and interesting applications.

The _Yubikey Handbook_ is an attempt of exploring those use cases and is intended to be a living document. It is focused on the Yubikey 4/Yubikey 4 Nano. With some adaptations, parts of this document will also apply to the Yubikey NEO.

Some cryptography knowledge is expected as not all steps are detailed thoroughly.

## Personal Identity Verification (PIV)

Yubikey supports the Personal Identity Verification (PIV or FIPS 201) card interface (NIST SP 800-73), enabling RSA or ECC sign/encrypt/decrypt operations using a private key stored on the SmartCard through common interfaces such as PKCS#11.

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

**Slot 9e: Attestation**

Attests other slot keys were generated on the device.

### Yubikey PIV Manager

The [Yubikey PIV Manager](https://developers.yubico.com/Yubikey-piv-manager) is a GUI tool to manage PIV-related data on a Yubikey. The command-line version can be installed via Homebrew:

```
❯ brew install yubico-piv-tool
```

The command-line utility is more powerful (e.g. allows overriding pin and touch default slot policies) but it also more prone to user errors. It's very easy to overwrite an existing key, for instance.

After [downloading](https://developers.yubico.com/Yubikey-piv-manager) and installing Yubikey PIV Manager and `yubico-piv-tool`, insert your Yubikey. You will be prompted [initialize your device](#device-initialization).

## Device initialization

Device initialization is straightforward but requires some organization around secret management. In the future, this can be improved by defining a group policy distributed via MDM which can enforce some of the settings mentioned below.

1. Enter a new PIN with 8 numeric characters if macOS login is intended. macOS won't work if the PIN contains alphanumeric characters. Generate and store this PIN securely on a password manager.
2. Set the _Management Key_ option to _Use a separate key_.
3. Under _Store management key_, randomize and store the resulting key on a password manager.
4. Enter a new PUK with 8 alphanumeric characters (A-Z, a-z, 0-9 and symbols are allowed), also generated on a password manager.
5. When asked if you want to _Set up Yubikey for macOS_ by generating certificates, choose _No_. This can be handled later on more selectively.

## Authenticating SSH with PIV and PKCS#11 (client)

One of the coolest features of the Yubikey is authenticating SSH sessions via PKCS#11\. The private key is stored on the Yubikey and whenever it is accessed, Yubikey can require a touch action.

Besides the common remote login, all connections that use SSH, such as remote git server (e.g. GitHub), may trigger this behavior if desired. This is a protection on the client side to prevent unauthorized SSH private key access.

Note that only RSA keys are supported when using this method.

1. Create a 2048-bit RSA key pair:

  ```sh
  ❯ yubico-piv-tool -s 9a -a generate -k --pin-policy=once --touch-policy=always --algorithm=RSA2048 -o public.pem
  ```

  Enter Yubikey's _Management key_.

2. Create a self-signed certificate (or, alternatively, a certificate signing request):

  ```sh
  ❯ yubico-piv-tool -a verify-pin -a selfsign-certificate -s 9a -S '/CN=ssh/' --valid-days=365 -i public.pem -o cert.pem
  ```

  Enter PIN then touch the Yubikey.

  Alternatively replace `selfsign-certificate` by `request-certificate` and send the resulting `.csr` file for internal CA certification.

3. Import the (self-)signed certificate:

  ```sh
  ❯ yubico-piv-tool -k -a import-certificate -s 9a -i cert.pem
  ```

  Enter Yubikey's _Management key_.

4. Export the public key stored in the Yubikey in the correct format for OpenSSH:

  ```sh
  ❯ ssh-keygen -D /usr/local/opt/opensc/lib/pkcs11/opensc-pkcs11.so -e
  ```

  Alternatively, you can use the generated public key before you delete it:

  ```sh
  ❯ ssh-keygen -i -m PKCS8 -f public.pem
  ```

  You can now share this public key for SSH authentication (e.g `~/.ssh/authorized_keys`).

5. Check slot 9a status (optional):

  ```sh
  ❯ yubico-piv-tool -a status
  ```

6. Add the SSH key provided via PKCS#11 to the local ssh-agent:

  ```sh
  ❯ ssh-add -s /usr/local/opt/opensc/lib/pkcs11/opensc-pkcs11.so
  ```

  Enter the Yubikey PIN when it asks for the passphrase.

7. Confirm the key has been added:

  ```sh
  ❯ ssh-add -L
  ```

8. Alternatively, selectively add the PKCS11Provider to `~/.ssh/config`:

  ```sh
  Host github.com
    PKCS11Provider /usr/local/opt/opensc/lib/pkcs11/opensc-pkcs11.so
    Port 22
    User foobar
  ```

### Troubleshooting

**Could not add card "/usr/local/opt/opensc/lib/pkcs11/opensc-pkcs11.so": agent refused operation**

For the lack of a proper diagnostic, run `pkill ssh-agent` and physically remove and re-enter the Yubikey.

## Authenticating SSH via User Certificates (server)

A complicated aspect of security is reliability and guaranteeing the consistency of all security controls. Instead of relying on a central authentication authority such as LDAP or Kerberos, we can take advantage of SSH or, more specifically OpenSSH, to provide both.

In addition to [authenticating SSH client access with PIV and PKCS#11](#authenticating-ssh-client-access-with-piv-and-pkcs-11), it is possible to increment the security of the remote SSH authentication. Facebook and Yahoo have switched to SSH User Certificates to avoid lockdown if the central authentication system goes down. It also helps maintaining the `authorized_keys` file, as it does not scale well (it requires a 1:1 match).

A _SSH User Certificate Authority_ can sign and thus securely authenticate each client connecting to a server.

The signed certificate also designates the principals (login identities) that can be used with that certificate. For each user, the principals can be described on a file:

```sh
❯ mkdir /etc/ssh/auth_principals
❯ echo -e 'access-root' > /etc/ssh/auth_principals/root
❯ echo -e 'access-databases' > /etc/ssh/auth_principals/foobar
```

In this example, any signed certificate with the `access-root` principal would be allowed to SSH into that host with the `root` username, and any signed certificate with the `access-databases` principal would be able to login with the `foobar` user.

Now let's create the _SSH User Certificate Authority_.

1. Using an air gapped computer, generate the user certificate authority:

  ```sh
  ❯ ssh-keygen -C "SSH User Certificate Authority" -f sshuser.root.ca
  ```

2. Distribute the public key (`sshuser.root.ca.pub`) to `/etc/ssh/` on every host. Make sure the file is `chmod 644`.

3. Update `/etc/ssh/sshd_config` to include the new CA and principals file:

  ```sh
  TrustedUserCAKeys /etc/ssh/sshuser.root.ca.pub
  AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u
  ```

4. Have the user/client extract the public key from their Yubikey so it can be signed by the new CA on the air gapped computer:

  ```sh
  ssh-keygen -D /usr/local/opt/opensc/lib/pkcs11/opensc-pkcs11.so -e
  ```

5. Sign the user certificate on the air gapped computer, with particular attention to the login name (`<user>`), the principals which this certificate will be able to claim (`<principals>`, separated by commas), the certificate expiration time (`+52w`) and the serial number (`<serial>`, an integer which should be tracked):

  ```sh
  ❯ ssh-keygen -s sshuser.root.ca -I <user> -n <principals> -V +52w -z <serial> <user>.pub

  Signed user key foobar-cert.pub: id "foobar" serial 1928121 for access-root valid from 2016-12-10T00:10:00 to 2017-12-09T00:10:10
  ```

6. Confirm the user certificate looks good:

  ```sh
  ❯ ssh-keygen -Lf <user>-cert.pub

  user-cert.pub:
      Type: ssh-rsa-cert-v01@openssh.com user certificate
      Public key: RSA-CERT SHA256:NWmw3siRlxn3bsIhzaFrCsh66KKIWapFuZsNiDXhRLw
      Signing CA: RSA SHA256:HLD1Eb4XiCoyXew23skyisJt+3P02MOsrHHbK/DmlgY
      Key ID: "foobar"
      Serial: 1928121
      Valid: from 2016-12-10T00:10:00 to 2017-12-09T00:10:10
      Principals:
              access-root
      Critical Options: (none)
      Extensions:
              permit-X11-forwarding
              permit-agent-forwarding
              permit-port-forwarding
              permit-pty
              permit-user-rc
  ```

7. Copy `<user>-cert.pub` to the client's `~/.ssh` directory and name it `id_rsa-cert.pub`. The name is quite specific, as there seems to be a limitation on `opensc-pkcs11` to detect a certificate other than `id_rsa-cert.pub`.

### Generating the Key Revocation List (KRL)

The KRL is a compact binary format which allows revoking SSH signed certificates.

1. Create an empty revoking list:

  ```sh
  ❯ touch /etc/ssh/revoked_keys
  ```

2. Update `/etc/ssh/sshd_config` to include the new Key Revocation List:

  ```sh
  ❯ RevokedKeys /etc/ssh/revoked_keys
  ```

3. When necessary, revoke the first signed certificate:

  ```sh
  ❯ ssh-keygen -k -f revoked_keys -s sshuser.root.ca.pub foo-cert.pub
  ```

4. When necessary, append more revoked certificates (using `-u`):

  ```sh
  ❯ ssh-keygen -k -f revoked_keys -s sshuser.root.ca.pub -u bar-cert.pub
  ```

5. Confirm that revocation worked:

  ```sh
  ❯ ssh-keygen -Qf revoked_keys foo-cert.pub
  ```

6. Distribute the updated `revoked_keys` to every host (`/etc/ssh/revoked_keys`) using `rsync`, `scp` or other orchestration utility.

NOTE: `ssh-keygen` should not require the signed public certificate to revoke it. Instead, using just the serial number should work. However, this is currently not working on OpenSSH 7.2p2 (Ubuntu).

## Authenticating SSH Host Certificates (client)

Similar to the [SSH User Certificates story](#remote-ssh-authentication-via-user certificates), it is also possible to authenticate hosts via client side certificate authority authentication. An SSH Server Certificate Authority signs server certificates and the client only needs to be aware of the CA's public key.

1. Using an air gapped computer, generate the server certificate authority:

  ```sh
  ❯ ssh-keygen -C "SSH Server Certificate Authority" -f sshserver.root.ca
  ```

2. Sign the host public key using the server certificate authority:

  ```sh
  ❯ ssh-keygen -s sshserver.root.ca -I <identity> -h -n <hostname> -V +52w /etc/ssh/ssh_host_rsa_key.pub

  Signed host key /etc/ssh/ssh_host_rsa_key-cert.pub: id "foobar.com-key" serial 0 for foobar.com valid from 2016-12-10T00:10:00 to 2017-12-09T00:10:10
  ```

3. Update `/etc/ssh/sshd_config` to include the new host certificate:

  ```sh
  HostCertificate /etc/ssh/ssh_host_rsa_key-cert.pub
  ```

4. Add the certificate authority to the local ssh client (`~/.ssh/known_hosts`):

  ```
  @cert-authority *.foobar.com <content of sshserver.root.ca.pub>
  ```

### Additional resources

- [How I've set up SSH keys on my Yubikey 4 (so far)](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/Yubikey4ForSSHKeys)
- [How to Harden SSH with Identities and Certificates](https://ef.gy/hardening-ssh)
- [Scalable and secure access with SSH](https://code.facebook.com/posts/365787980419535/scalable-and-secure-access-with-ssh/)

## 2FA via Yubico OTP (server)

Improving the security of the client (authenticating SSH with PIV) and simplifying the managing tasks on the server (authenticating SSH via User Certificates) is the first step for a more secure environment. However, they still rely on a single factor for authentication. This can be changed to require a second factor (2FA).

Enabling 2FA with SSH or `sudo` on a remote server requires online validation, either by using YubiCloud's OTP Validation service backed by their own YubiHSM (Hardware Security Module), or hosting it elsewhere inside a private network.

The main benefit of running the OTP validation service on a private server is that the AES keys programmed into the Yubikeys are in complete control. YubiCloud, on the other hand, manages those AES keys for us, so it has the benefit of providing a seamless, no-maintenance experience.

YubiCloud offers a premium service with statistics (personalized monthly report via email, server uptime and Yubikey usage reports), including an SLA for an undisclosed price.

Other means of offline validation, such as HMAC-SHA1 Challenge-Response or U2F, are not supported at the moment due to architectural constraints (HMAC-SHA1 Challenge-Response would require the Yubikey to be plugged-in on the remote server) or lack of software support ([OpenSSH does not support the U2F standard yet](https://bugzilla.mindrot.org/show_bug.cgi?id=2319)).

### Setting up a remote server

In the following example Docker is used to create an Ubuntu-based server where Yubico software will be installed. It uses the latest Ubuntu distribution version available on Docker Hub.

#### Prerequisites (demonstration only)

You will need a Linux-based box like Ubuntu running OpenSSH:

```sh
❯ docker run --name yubico-ubuntu -p 2222:22 -it ubuntu
```

Inside the Docker container:

```sh
❯ apt-get update
❯ apt-get install -y openssh-server vim
❯ mkdir -p /root/.ssh
```

Add a user for testing purposes:

```sh
❯ adduser foobar
❯ usermod -G sudo foobar
❯ mkdir -p /home/foobar/.ssh
```

Add your public ssh key to both users. First, locally:

```sh
❯ cat ~/.ssh/id_rsa.pub | pbcopy
```

Then:

```sh
❯ echo "<pubkey>" >> /home/foobar/.ssh/authorized_keys
❯ echo "<pubkey>" >> /root/.ssh/authorized_keys
```

If you'd like to ssh with the `root` user, remember to add a password as that will be required later on:

```sh
❯ passwd root
```

#### Configure OpenSSH (sshd) for 2FA authentication

Enable strong 2FA authentication by updating the `/etc/ssh/sshd_config` with the following changes:

```sh
# Require public key *and* password authentication. Without this, a valid public
# key would bypass the Yubikey requirement.
AuthenticationMethods publickey,password

# Enable the password authentication backend.
PasswordAuthentication yes

# Disable the keyboard-interactive mode which could be used to ask for the
# password.
ChallengeResponseAuthentication no

# Enable PAM integration for authentication as this is the system that Yubikey
# integrates with.
UsePAM yes
```

If you want to login with the _root_ user via ssh, add or update the `PermitRootLogin` under the same file, replace `prohibit-password` by `yes`:

```sh
# Enable root login via ssh.
PermitRootLogin yes
```

Restart `ssh`. Note that if you're already inside an ssh session, you won't be disconnected.

```sh
❯ service ssh restart
```

#### Install libpam-yubico

The package `libpam-yubico` provides the requires libraries to integrate Yubico's software into PAM (Linux Pluggable Authentication Modules).

Add the PPA and install the package:

```sh
❯ apt-get install -y vim software-properties-common
❯ add-apt-repository -y ppa:yubico/stable
❯ apt-get update
❯ apt-get install -y libpam-yubico
```

#### Create the Yubikey PAM authentication policy

##### Obtaining the Yubikey token ID

The Yubikey token ID is a public identifier that uniquely identifies it. You can obtain the Yubikey token ID in several ways.

The quickest way of getting the token ID is to remove the last 32 characters of any OTP (One Time Password) generated by the Yubikey.

1. Open a terminal.
2. Long-touch the Yubikey.
3. An OTP token will be output to the shell:

```sh
❯ cccccccgklgcvnkcvnnegrnhgrjkhlkfhdkclfncvlgj
bash: cccccccgklgcvnkcvnnegrnhgrjkhlkfhdkclfncvlgj: command not found
```

The corresponding token ID will be `cccccccgklgc`:

```
cccccccgklgcvnkcvnnegrnhgrjkhlkfhdkclfncvlgj
┊←       →┊┊←            32              →┊
```

If you'd like to experiment with other ways, you can activate the debug mode of a local Yubico PAM module and when authenticating with it. The ID will be printed out in the debug information. There is also a [https://demo.yubico.com/modhex.php](modhex converter web tool) where you can enter the token above, select _OTP_ as the source format and get the token ID as the _Modhex encoded_ value.

##### Create system-wide Yubikeys mapping

The file `/etc/yubikeys` (as listed below), must contain the UNIX user name of the remote server and the Yubikey token ID separated by colons for each user. Example format:

```sh
<user-1>:<yubikey-id-1>
<user-2>:<yubikey-id-1>
<user-3>:<yubikey-id-2>:<yubikey-id-3>
```

So, continuing the example above and allowing the same Yubikey to authenticate with two different users:

```sh
root:cccccccgklgc
foobar:cccccccgklgc
```

Edit, then:

```sh
❯ chmod 644 /etc/yubikeys
```

##### Using individual Yubikey user mapping

Alternatively, an individual mapping file can be configured per user. In that case, the `authfile` directive should be removed.

First, create a `.yubico` folder inside the user's home:

```
❯ mkdir -m700 /home/<user>/.yubico
```

Then, add the Yubikey mapping to `/home/<user>/.yubico/authorized_yubikeys`:

```
<user>:<yubikey-id-1>
```

#### Yubikey authentication module

##### Create an API key for YubiCloud

Grab a free API key from [Yubico](https://upgrade.yubico.com/getapikey/). One application, context or server should mean a new client id.

Enter your email, select the `Yubikey OTP` input and touch your Yubikey. The resulting page will include:

```
Client ID: <clientId>
Secret Key: <secretKey>
```

Store these credentials in a safe place (password manager) as you will need them in the next step.

##### Create the Yubikey authentication module

After the system-wide mapping is completed and the API credentials generated, create and add the following configuration line to `/etc/pam.d/yubikey-auth`:

```sh
# Enable YubiCloud OTP Validation (implicitly includes `mode=client` and the default validation `urllist`).
auth    required        pam_yubico.so id=<clientId> key=<secretKey> authfile=/etc/yubikeys
```

To enable debugging, create the log files and add the `debug` and `debug_file` parameters as arguments to `pam_yubico.so`:

```sh
❯ touch /var/log/yubikey-auth.log
❯ chmod go+w /var/log/yubikey-auth.log
```

Update the `/etc/pam.d/yubikey-auth` file:

```sh
auth    required        pam_yubico.so id=<clientId> key=<secretKey> authfile=/etc/yubikeys debug debug_file=/var/log/yubikey-auth.log
```

Remove when debugging is no longer necessary.

##### Update _common-auth_ authentication module

Add the `try_first_pass` directive to the `pam_unix.so` backend inside the `/etc/pam.d/common-auth` module so that it first attempts to use the previous stacked module's password in case that satisfies this module as well before asking it again. Not all distributions have a `common-auth` file, so you may look on other files to see what makes sense.

Also, you must include the previously created Yubikey authentication module (order matters):

```sh
auth    include                         yubikey-auth
auth    [success=1 default=ignore]      pam_unix.so nullok_secure try_first_pass
```

### Testing

If you've enabled `root` login via ssh, you should be able to login in the Docker container using the published `2222` ssh port:

```sh
❯ ssh root@127.0.0.1 -p 2222
Authenticated with partial success.
root@127.0.0.1 password:
```

You should see the _Authenticated with partial success_ text, which means that the authentication against the public key succeeded.

Now, you must enter the user's password and, **without** hitting enter. Long-touch the Yubikey until a newline is entered automatically.

If you consider the password `foobar` for the `root` user, the actual password that will get sent is:

```
root@127.0.0.1 password: foobarcccccccgklgcvnkcvnnegrnhgrjkhlkfhdkclfncvlgj
```

`libpam-yubico` will remove the characters pertaining to the OTP, send it to YubiCloud, and upon success forward the remaining characters to the next PAM module (in this case, `pam_unix.so`) validate the user password.

After 2-3s, you should be logged in! Now, exit and login with `foobar`. Attempt to escalate privileges by doing `su root` and you will see that the Yubikey for the `root` user will be required (the same principle applies - first enter the password followed by the long-touch on the Yubikey).

As you may have noticed that during SSH, there are actually three factors involved, not two - public key authentication, password and Yubikey OTP. This is actually a limitation of OpenSSH, as public key authentication plus Yubikey OTP without requiring the user's UNIX password is not possible at the moment.

## OATH (TOTP and HOTP)

The Initiative for Open Authentication (OATH) is responsible for developing two standards - TOTP (clock-based) and HOTP (counter-based). Both are used [extensively](https://twofactorauth.org/) nowadays.

The Yubikey can emit an HOTP token when touched. For TOTP, a companion application ([Yubico Authenticator](https://developers.yubico.com/yubioath-desktop)) must be used as Yubikeys do not have an internal clock.

**Why use a Yubikey for OATH**?

- The shared secrets are stored securely in the Yubikey.
- Can be used on any computer and thus is not conditioned by typical mobile device issues, such as drained battery.

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

## U2F (Security Keys)

U2F is an open authentication standard based on public key cryptography created by the FIDO Alliance for accessing web-based services. Authentication is done on the client by creating a new key pair for each service and then providing its public key. During authentication, the client proves it is in the possession of the private key by signing a challenge sent by the provider.

No codes are manually typed or copied during this exchange, making U2F more convenient and also less susceptible to phishing and MITM attacks. Simply touch the Yubikey to approve signing the challenge.

Recent versions of Chrome and Opera come with U2F supported enabled by default. Support for Safari can be achieve using the [Safari-FIDO-U2F](https://github.com/blahgeek/Safari-FIDO-U2F) extension, although some user-agent emulation (`Develop > User Agent > Chrome for Mac`) is still required.

When registering the Yubikey as a Security Key, a good alias may be `Yubikey (<serial number in decimal format>)`.

[Some of the projects](http://www.dongleauth.info) that support U2F:

- Google
- GitHub
- Dropbox
- Sentry

There is a great article _[Security Keys: Practical Cryptographic Second Factors for the Modern Web](http://fc16.ifca.ai/preproceedings/25_Lang.pdf)_ from Google detailing their experience deploying more than 50,000 Security Keys to their employees.

## Docker Content Trust

Docker Content Trust allows delivering trusted images over in insecure network using signed containers. Docker Engine can verify the integrity and freshness of an image along the entire Docker flow (`push`, `pull`, `build`, `create` and `run` operations). It is commonly referred as an opinionated integration of Notary in Docker.

Notary is a tool based on [The Update Framework](http://theupdateframework.com/) that solves the problem of secure software updates delivery over the network. It provides mitigations for the following known attacks:

- Image tampering, by digitally signing each layer
- Key compromise, by providing transparent key rotation mechanisms out of the box
- Replay attacks, by using a timestamped key that ensures content freshness

Docker Hub runs a Notary service against which official signed images can be verified. When running a private registry (e.g. Amazon ECR), a private Notary service is required.

### Key management

Content trust is directly associated with an image tag and each repository has a set of keys that publishers use to sign each image.

A repository can have both unsigned and signed images. They live as separate entities, so the same tag (e.g. `latest`) can point to different contents depending on whether Docker Content Trust is enabled or not on the client.

Image trust builds on 4 keys:

- A `root` key (_offline_) which is the root anchor of the content trust for an image. This is key that gets stored on the Yubikey and only brought online for a limited number of operations
- A `targets` key (_online_) - the key that signs the actual files downloaded, stored on the client and encrypted at rest
- A `snapshot` key (_online_), which signs the metadata file containing information about all the other metadata available on the collection
- A `timestamp` key (_online_), which ensures content freshness by periodically signing a timestamped statement.

The `snapshot` and the `timestamp` can be managed by the Notary service for convenience.

### Running Notary services

In an organization, a internal Notary HA deployment is expected to be available in order for Docker Content Trust to be usable. Hence, this topic assumes familiarity with the [Notary service architecture](https://github.com/docker/notary/blob/master/docs/service_architecture.md).

Notary ships with support for multiple storage backends. Choose one appropriate to your infrastructure and team.

Both `notary-server` and `notary-signer` should be behind a load balancer. The public facing entry is `notary-server` and only serves public data. The `notary-signer` stores the timestamp keys online - it can be compared to an HSM.

The content publisher owns the client side key and is the one responsible for publishing new content.

#### Configuring Notary

Depending on the environment and purpose of running Notary services, there are two options: using `docker-compose` when running locally or running each service separately, usually through an orchestration layer (Kubernetes, Rancher, Swarm and so on). Configuring the latter is outside the scope of this document, while the former should only be used for demonstration purposes.

The following examples assume you will run Notary on your internal infrastructure and store Docker images on Amazon ECR.

##### Using docker-compose

The following examples assume that `postgres` was the choice of your `backend`.

Start by cloning the official repository:

```sh
❯ git clone https://github.com/docker/notary.git
❯ git checkout v0.5.0
```

You can define the configuration via environment variables on the `docker-compose.postgres.yml` file that will override config settings. Alternatively you can also edit `fixtures/server-config.postgres.json` and `fixtures/signer-config.postgres.json`. If you choose to go with the default, no change is required.

Note that if you choose to use `fixtures/regenerateTestingCerts.sh` for local testing with `Docker for Mac.app` using Docker-in-Docker for simulating multiple clients, make sure to add `IP:192.168.65.1` to the `subjectAltName` or `192.168.65.1 notaryserver` to `/etc/hosts`, otherwise any Docker operation will result in a invalid certificate error.

If you have bootstrapped Notary before, you may need to delete its volume to get the new certificates reloaded (`docker volume rm notary_notary_data`). Beware that this will destroy all previously stored testing data.

```sh
docker-compose -f docker-compose.postgresql.yml up
```

If you chose not to sign the certificates using a previously-trusted internal CA, you will need to manually trust the `fixtures/root-ca.crt`, otherwise Docker will throw out this error when interacting with Notary:

```sh
ERRO[0000] could not reach https://127.0.0.1:4443: Get https://127.0.0.1:4443/v2/: x509: certificate signed by unknown authority
```

Trust the self-signed Notary's Intermediate CA:

```sh
❯ mkdir -p ~/.docker/tls/127.0.0.1:4443/
❯ cp fixtures/intermediate-ca ~/.docker/tls/127.0.0.1:4443/
```

From now on, Docker can be instructed to always use the private Notary server:

```sh
❯ export DOCKER_CONTENT_TRUST=1
❯ export DOCKER_CONTENT_TRUST_SERVER=https://127.0.0.1:4443
```

A completely separate environment can be created by using Docker-in-Docker, where delegations can be explored:

```sh
❯ docker run -d --privileged -e DOCKER_CONTENT_TRUST=1 -e DOCKER_CONTENT_TRUST_SERVER=https://notaryserver:4443 docker:dind
905c2a005f34745175299da9652a991860f965bbd1e24642e20c6abb9de03174
```

Then enter the container:

```sh
❯ docker exec -it 905c2a005f34745175299da9652a991860f965bbd1e24642e20c6abb9de03174 sh

/ # echo "192.168.65.1 notaryserver" >> /etc/hosts
/ # wget -O /usr/local/bin/notary https://github.com/docker/notary/releases/download/v0.4.2/notary-Linux-amd64 && chmod a+x /usr/local/bin/notary
```

Make sure to add the Notary server certificate to `/root/.docker/tls/notaryserver:4443/notaryserver.crt`.

If using Amazon ECR for a private registry and credentials are needed, echo them to a local file and pipe it to shell:

```
❯ cat aws.login | sh
```

#### Managing certificates

You should generate your own certificates before going to production.

Depending on the go version used, the `notary-server` certificate [may have](https://go-review.googlesource.com/#/c/10806/) to be marked for both EKUs `clientAuth` (for connection as a gRPC client) _and_ `serverAuth` (for serving TLS as a server).

#### Additional resources

- [Recommendations for deploying in production](https://github.com/docker/notary/blob/master/docs/running_a_service.md#recommendations-for-deploying-in-production)

### Pushing a signed Docker image

Now that the basic setup is done, let's push a signed Docker image.

#### Generate the root key on the Yubikey

1. List all the keys known on the host:

  ```sh
  ❯ notary -d ~/.docker/trust key list
  ```

  You probably don't have any keys available yet. Let's generate one.

2. Create an 256-bit ECC key pair:

  ```sh
  ❯ yubico-piv-tool -s 9c -a generate -k --pin-policy=always --touch-policy=always --algorithm=ECCP256 -o public.pem
  ```

  Enter Yubikeys _Management key_.

3. Create a self-signed certificate (or, alternatively, a certificate signing request):

  ```sh
  ❯ yubico-piv-tool -a verify-pin -a selfsign-certificate -s 9c -S '/CN=root/' --valid-days=365 -i public.pem -o cert.pem
  ```

  Enter PIN then touch the Yubikey.

  Alternatively replace `selfsign-certificate` by `request-certificate` and send the resulting `.csr` file for internal CA certification.

  The `CN=root` is what allows Notary to find the key on the Yubikey, so it should be kept that way.

4. Import the (self-)signed certificate:

  ```sh
  ❯ yubico-piv-tool -k -a import-certificate -s 9c -i cert.pem
  ```

  Enter Yubikey's _Management key_.

You can choose to generate the private key outside the Yubikey, in case you prefer to have a local backup copy. `notary key generate` will generate a private key locally and then find an empty slot to import it on the Yubikey.

When Notary asks for the _SO PIN_, enter the Yubikey's _Management Key_.

You should now have the `root` key available:

```sh
❯ notary -d ~/.docker/trust key list

ROLE    GUN    KEY ID                                               LOCATION
----    ---    ------                                               --------
root           bf98cc496cb05fd2b88b01d3200900ff05ec83a1f3690690f…   Yubikey
```

#### Pushing the image

Now that a `root` key is available, it's time to initialize the repository on the first push.

Consider this as your app:

```docker
FROM alpine

RUN true
```

Make sure you have all trusted metadata using the official Notary server when building the image by temporarily redefining the content trust server:

```sh
❯ DOCKER_CONTENT_TRUST_SERVER=https://notary.docker.io docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.0 .
```

Then push it upstream, forcing Docker to initialize the signed repository:

```
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.0

The push refers to a repository [<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Pushed
1.0.0: digest: sha256:475d897467451caf22f22ad9fd2856a5dd4a876b9eb2daab4d474185f4244e8d size: 2101
Signing and pushing trust metadata
Enter the User Pin for the attached Yubikey:
Please touch the attached Yubikey to perform signing.
Enter passphrase for new repository key with ID 9c738a6 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Repeat passphrase for new repository key with ID 9c738a6 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Enter the User Pin for the attached Yubikey:
Please touch the attached Yubikey to perform signing.
Finished initializing "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app"
Successfully signed "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.0
```

The repository key passphrase should be generated and stored by a password manager. This will be a personal key, i.e., not intended to be shared. It will be stored, in its encrypted form, under `~/.docker/trust`. The repository key is will be generated with the `targets` role.

```
❯ notary -d ~/.docker/trust key list
```

Now edit the file and generate a new build:

```docker
FROM alpine

RUN true
RUN uname
```

Build it:

```sh
❯ docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.1 .
```

And push it:

```sh
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.1

The push refers to a repository [<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Layer already exists
1.0.1: digest: sha256:afc214501f950ca11246ceb62fb5c07dd5856f359d6122a620e2c60071a484bc size: 2531
Signing and pushing trust metadata
Enter passphrase for repository key with ID 9c738a6 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Successfully signed "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.1
```

On the second push, only the passphrase of the repository key was required.

That's it. Signed builds using an offline root key (Yubikey) and one online repository key.

### Listing signed images on a remote repository

Let's list the existing images available on a private registry using a private Notary server:

```sh
❯ notary -s https://127.0.0.1:4443 -d ~/.docker/trust list <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

NAME     DIGEST                                                              SIZE (BYTES)    ROLE
----     ------                                                              ------------    ----
1.0.0    475d897467451caf22f22ad9fd2856a5dd4a876b9eb2daab4d474185f4244e8d    2102            targets
1.0.1    afc214501f950ca11246ceb62fb5c07dd5856f359d6122a620e2c60071a484bc    2537            targets
```

This also works on the official Docker Hub registry by pointing to the public Notary service (you'll have to prepend `docker.io/library`):

```sh
❯ notary -s https://notary.docker.io -d ~/.docker/trust list docker.io/library/alpine
```

### Delegation roles

Delegation roles allows assigning signing privileges to other repository collaborators. This is particularly interesting in organizations that have multiple people working on the same repository.

#### Generating a delegation key

Make sure you're using OpenSSL 1.0.2+, otherwise the keys will be signed with SHA1, which Notary will refuse to import (_invalid SHA1 signature algorithm_):

```sh
❯ brew install openssl
```

Make sure the `openssl` in your `$PATH` points to the correct version:

```sh
❯ openssl version
OpenSSL 1.0.2j  26 Sep 2016
```

Have the collaborator generate a new key pair on their machine (you can use the Docker-in-Docker container for this):

1. Create an 256-bit ECC key pair:

  ```sh
  ❯ openssl ecparam -name prime256v1 -genkey -out delegation.key -noout
  ```

2. Create a certificate signing request:

  ```sh
  ❯ openssl req -new -sha256 -key delegation.key -out delegation.csr
  ```

3. Self-sign the certificate (or, alternatively, sign the csr using an internal CA):

  ```sh
  ❯ openssl x509 -req -days 365 -in delegation.csr -signkey delegation.key -out delegation.crt
  ```

4. Import the private key:

  ```
  ❯ notary -v -s https://127.0.0.1:4443 -d ~/.docker/trust key import delegation.key --role user --gun <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

  Enter passphrase for new user key with ID e93a684 (tuf_keys):
  Repeat passphrase for new user key with ID e93a684 (tuf_keys):
  ```

  Enter a passphrase generated and stored on a password manager.

5. Send the public key (`delegation.crt`) to the repository owner.

#### Importing a delegation certificate

As a repository owner:

1. Add the delegation key to the repository using the `targets/releases` path, as this is what Docker searches for when signing an image (first `targets`, then `targets/releases`:

  ```sh
  ❯ notary delegation -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust add <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app targets/releases delegation.crt --all-paths

  Addition of delegation role targets/releases with keys [e93a68479026f002b9dedb35f563cf5abc50aecd18b2205ad296d3101c0d3c21], with paths ["" <all paths>], to repository "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app" staged for next publish.
  ```

2. Check the unpublished (staged) changes:

  ```sh
  ❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust status <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

  Unpublished changes for <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:

  #    ACTION    SCOPE               TYPE          PATH
  -    ------    -----               ----          ----
  0    create    targets/releases    delegation
  1    create    targets/releases    delegation
  ```

3. Publish the queued changes:

  ```sh
  ❯ notary -v -s https://127.0.0.1:4443 -d ~/.docker/trust publish <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

  Pushing changes to <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app
  Enter passphrase for targets key with ID 79a6fca:
  Successfully published changes for repository <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app
  ```

  Enter the previously created `targets` repository key passphrase.

4. Confirm the delegation list looks correct:

  ```
  ❯ notary -v -s https://127.0.0.1:4443 -d ~/.docker/trust delegation list <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app

  ROLE                PATHS             KEY IDS                    THRESHOLD
  ----                -----             -------                    ---------
  targets/releases    "" <all paths>    61a6430…                   1
  ```

#### Using a delegation key

The collaborator can now push to the repository using Docker Content Trust. Docker will automatically choose and pick the right key for the `targets/release` role.

Edit the file on the Docker-in-Docker container:

```docker
FROM alpine

RUN true
RUN uname
RUN echo collaborating
```

Build the new image:

```sh
❯ DOCKER_CONTENT_TRUST_SERVER=https://notary.docker.io docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3 .
```

Push the new image:

```sh
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3

The push refers to a repository [<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Pushed
1.0.3: digest: sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268 size: 2980
Signing and pushing trust metadata
Enter passphrase for repository key with ID e93a684 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Successfully signed "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.3
```

Test on the repository owner side that the image signed by the collaborator is valid:

```sh
❯ docker pull <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3
Pull (1 of 1): <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3@sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268
sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268: Pulling from app
3690ec4760f9: Already exists
Digest: sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268
Status: Downloaded newer image for <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app@sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268
Tagging <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app@sha256:71482bc2bcf58d113dd109d944749707580b0ea7bb76df81624b68e4d0834268 as <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.3
```

Notice that the digest from the collaborator matches the one received on the owner side.

Now attempt to edit the Dockerfile on the owner side again:

```docker
FROM alpine

RUN true
RUN uname
RUN date
```

And build it:

```sh
❯ docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.4 .
```

Everything looks good. Now try to push it:

```sh
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.4

The push refers to a repository [<aws_account_id>.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Pushed
1.0.4: digest: sha256:19cbb30c36b9855aff3ccf7b052bbf6032b7acf4510ea311e82a2e51d926fd8d size: 2966
Signing and pushing trust metadata
Failed to sign "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.4 - no valid signing keys for delegation roles
no valid signing keys for delegation roles
```

What happened here was that when delegation was enabled for this repository, Docker now requires keys to be valid under the `targets/releases` role. Remember that the original key, created upon repository initialization (first push), was listed with the `targets` role instead.

So in order to enable the repository owner to also be able to sign images, the owner needs to follow the exact same steps as all collaborators, i.e., creating and adding its owner `targets/release` key to the repository.

While following the collaborator instructions, you may get this error if you have your Yubikey plugged in when running `notary key import`:

```
ERRO[0007] failed to import key to store: yubikey only supports storing root keys, got user for key: 6965a1ee8ff68a211d769243c0b171f90cb03a337d2337cc91650b843a5bc1ff
```

When the import command is ran, Notary assumes that if a Yubikey is plugged in, it should copy the private key there too. However, a Yubikey should only be used for `root` keys, so when attempting to import a `user` key, it throws out this harmless error. In the future, it will likely be ignored.

After you've imported the key, the resulting list should be:

```sh
❯ notary -d ~/.docker/trust key list

ROLE       GUN                          KEY ID                                                              LOCATION
----       ---                          ------                                                              --------
root                                    bf98cc496cb05fd2b88b01d3200900ff05ec83a1f3690690f2c9341976b64728    yubikey
user                                    a726c2f62f2239055b7a1881c12d0de636b62e0a2c1ef21044083c51962f1959    ~/.docker/trust/private
targets    ...st-1.amazonaws.com/app    9c738a648878fab6124f70f78879dc1da89bae6ac0574c0ea6dfa6f20e80816c    ~/.docker/trust/private
```

Continue with the delegation key steps, adding the new to the delegation and publishing the changes. You will be asked to enter your original `targets` key, just like when adding the first collaborator key.

After you're done with your own delegation key, re-issue the push command:

```sh
❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:1.0.4
The push refers to a repository [<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app]
011b303988d2: Layer already exists
1.0.4: digest: sha256:19cbb30c36b9855aff3ccf7b052bbf6032b7acf4510ea311e82a2e51d926fd8d size: 2966
Signing and pushing trust metadata
Enter passphrase for user key with ID a726c2f:
Successfully signed "<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app":1.0.4
```

Notice that the passphrase is for your own delegation key now and the push finally works.

#### Automating image signing on CI systems

1. Create a delegation key for the CI system.
2. Expose an encrypted passphrase in the CI environment that imports the key via the local notary client:

  ```sh
  ❯ export NOTARY_DELEGATION_PASSPHRASE=foobar
  ❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust key import ./delegation.key --role user
  ```

3. Use the encrypted passphrase to sign and push the image:

  ```sh
  ❯ export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE=$NOTARY_DELEGATION_PASSPHRASE
  ❯ DOCKER_CONTENT_TRUST_SERVER=https://notary.docker.io docker build -t <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:latest .
  ❯ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app:latest
  ```

### Removing a delegation key

As a repository owner, remove a key from all delegation roles:

```sh
❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust delegation purge  <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app --key 900b53cd7116e0eda33905ad9f446b93a4620b6762597bebb5ec529ec842b611
```

Or, optionally, you can remove from a single role:

```sh
❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust delegation remove  <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app targets/releases 900b53cd7116e0eda33905ad9f446b93a4620b6762597bebb5ec529ec842b611
```

### Rotating a key

Transparent key rotation is at the heart of Notary. It enables a rapid response to a compromised key scenario.

#### Snapshot key

To rotate the `snapshot` key:

```sh
❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust key rotate <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app snapshot -r

Enter passphrase for new targets key with ID 3c10139 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Repeat passphrase for new targets key with ID 3c10139 (<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app):
Enter the User Pin for the attached Yubikey:
Please touch the attached Yubikey to perform signing.

Successfully rotated targets key for repository <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app
```

#### Timestamp key

The `timestamp` key can also be rotated:

```sh
❯ notary -D -v -s https://127.0.0.1:4443 -d ~/.docker/trust key rotate <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app timestamp -r

Enter the User Pin for the attached Yubikey:
Please touch the attached Yubikey to perform signing.

Successfully rotated timestamp key for repository <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app
```

This will require removing the `timestamp.json` file on each collaborator machine:

```
rm ~/.docker/trust/tuf/<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app/metadata/timestamp.json
```

#### Targets key

TBD to avoid _Warning: potential malicious behavior - trust data has insufficient signatures for remote repository
<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/app: valid signatures did not meet threshold_</aws_account_id>

### Threshold validation signing

One of the most exciting features that Docker Content Trust will enable in the future is the concept of _threshold validation signing_, which will allow staged verification signing. This will enable verification pipelines such as making sure that an image can only be deployed to staging after being signed by the CI system, or that an image can only be deployed to production once certain subset of keys is present on the image's signature (user key, CI key, staging key and QA key).

There will also be a possibility of defining signing thresholds within a single role (i.e. requiring just one 1 out of 5 CI keys, 2 out of 4 QA keys, etc).

Discussion is actively happening on GitHub:

- [Implement Threshold Signing mechanisms on the client/server](https://github.com/docker/notary/issues/841)
- [TAP3](https://github.com/theupdateframework/taps/blob/master/tap3.md)
- [TAP4](https://github.com/theupdateframework/taps/blob/master/tap4.md)

## OpenPGP

OpenPGP is an open standard for encrypting and signing. Although there has been some [recent](https://www.yubico.com/2016/05/secure-hardware-vs-open-source/) [debate](https://www.reddit.com/r/linux/comments/4ls94a/yubico_has_replaced_all_opensource_components/) [around it](https://www.reddit.com/r/linux/comments/5gw2e3/im_giving_up_on_pgp_filippo_valsorda/), it works very well with the Yubikey.

### Touch protection

The Yubikey 4 introduces a new touch feature

<sup>1</sup>

that enables a second layer of protection when using a private key stored on the device. The access will be conditioned by a user physically triggering the touch sensor, which detracts malware issuing command on the Yubikey without user knowledge.

The touch event is requested for up to 15 seconds, after which the Yubikey turns off the notification.

The touch sensor can be configured with the following parameters:

- **off**: touch is disabled
- **on**: touch is enabled
- **fix**: touch is enabled and can not be disabled unless a new private key is generated or imported.

Touch protection can be configured individually on each one of the GPG private keys and requires the use of the Admin PIN.

**References**

1. <https://developers.yubico.com/PGP/Card_edit.html>

#### Enabling touch protection

Install [https://developers.yubico.com/Yubikey-manager/](Yubikey-manager) on a permanent destination:

```sh
❯ brew install libu2f-host libusb swig ykpers
❯ git clone git@github.com:Yubico/Yubikey-manager.git
❯ git submodule update --init --recursive
❯ pip install -e .
```

The setup tools will automatically link the `ykman` binary to `/usr/local/bin/ykman` but the original git folder must remain on disk.

Then, enable touch protection for authentication (`aut`), encryption (`enc`) and signing (`sig`):

```sh
❯ ykman openpgp touch aut on
❯ ykman openpgp touch enc on
❯ ykman openpgp touch sig on
```

Confirm touch protection is enabled:

```sh
❯ ykman openpgp touch aut
Current touch policy of AUTHENTICATE key is ON.

❯ ykman openpgp touch enc
Current touch policy of ENCRYPT key is ON.

❯ ykman openpgp touch sig
Current touch policy of SIGN key is ON.
```

### Importing keys

GPG private keys can be copied or moved to the Yubikey. The following examples moves them.

```
❯ gpg --edit-key <keyId>

gpg> toggle
gpg> key 1
gpg> keytocard

Please select where to store the key:
   (1) Signature key
   (3) Authentication key
Your selection? 1

gpg> key 1
gpg> key 2
gpg> keytocard

Please select where to store the key:
   (2) Encryption key
Your selection? 2

gpg> key 2
gpg> key 3
gpg> keytocard

Please select where to store the key:
   (3) Authentication key
Your selection? 3

gpg> quit
```

Confirm the private keys have been moved to the Yubikey:

```sh
❯ gpg --list-secret-keys
```

If you see `ssb>`, it indicates a stub to the private key on Yubikey, which in this case means the move was successful.

Then, check the card status:

```sh
❯ gpg --card-status
```

### Editing metadata

Now that the private keys have been moved to the Yubikey, it's time to configure some basic properties, such as changing the default management PIN (what GPG calls the _Admin PIN_) and also the regular PIN whenever the private key is accessed.

```
❯ gpg --card-edit

gpg/card> admin
Admin commands are allowed

gpg/card> passwd
gpg: OpenPGP card no. … detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 3
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
PIN changed.

gpg/card> q

gpg/card> login
foobar

gpg/card> sex
m

gpg/card> name
Cardholder\'s surname: Bar
Cardholder\'s given name: Foo

gpg/card> lang
Language preferences: en

gpg/card> login
Login data (account name): foobar

gpg/card> list

gpg/card> quit
```

### Git signing

Both tags and commits can be signed by the Yubikey GPG keys.

#### Signing tags

Assuming you have an extra local file for adding gitconfig settings on your main `~/.gitconfig` file:

```
[include]
    path = .gitconfig.local
```

Configure `~/.gitconfig.local` to point to the correct GPG signing key of your interest:

```sh
[user]
    signingkey = <signingKeyId>
```

Enable `git tag -m <message>` to automatically force signed tags:

```
[tag]
    forceSignAnnotated true
```

Test if the signing key works by creating a new git tag on a temporary project:

```sh
❯ cd /tmp
❯ mkdir foo
❯ cd foo
❯ git init
❯ touch bar
❯ git add bar
❯ git ci -m "Add bar"
❯ git tag 1.0.0 -s -m "Release 1.0.0"
```

Enter the Yubikey GPG PIN and then touch it. Confirm the GPG signature was added to the tag:

```
❯ git show 1.0.0
```

#### Verifying tags

Import the public GPG key of the signer first and then use of any of the following commands to check for the `Good signature` message:

```sh
❯ git tag -v 1.0.0
❯ git cat-file -p 1.0.0
❯ git verify-tag 1.0.0
```

#### Signing commits

Continuing on the previous git repository, add a new file and commit it using the `-S` flag (not `-s` as that means `Signed-Off` on the commit command):

```
❯ touch biz
❯ git add biz
❯ git commit -S -m "Add biz"
```

You may enable auto-signing commits by adding the following to `~/.gitconfig.local`:

```
[commit]
    gpgSign = true
```

#### Verifying commits

Import the public GPG key of the signer first and then use of any of the following commands to check for the `Good signature` message:

```sh
❯ git show HEAD --show-signature
❯ git log --show-signature
❯ git verify-commit HEAD
```

#### Signing merges

`git merge` can be instructed to inspect and reject when merging a commit/branch that does not carry a trusted GPG signature with the `--verify-signatures` command.

If the branch being merged contains any commit that has not be signed and is valid, the merge will not proceed.

The merge commit itself can also be signed (using `-S`):

```
❯ git checkout -b enhancement/foo
❯ touch qux
❯ git add qux
❯ git commit -S -m "Add qux"
❯ git checkout master
❯ git merge --verify-signatures -S enhancement/foo
```

#### Signing pushes

`git push` can be instructed to sign the push. The server may use this to control the execution of certain hooks:

```
❯ git push --signed
```

Right now, GitHub does not appear to have support for signed pushes.

### Authenticating SSH with GPG

While technically it's certainly possible to authenticate SSH sessions using GPG, `gpg-agent` does not always have a friendly co-existence with `ssh-agent`.

Using PIV for authenticating SSH remains the recommended solution.

Enable `enable-ssh-support` and `write-env-file` under `~/.gnupg/gpg-agent.conf`:

```
default-cache-ttl 600
  max-cache-ttl 7200
  pinentry-program /usr/local/MacGPG2/libexec/pinentry-mac.app/Contents/MacOS/pinentry-mac
  enable-ssh-support
  write-env-file
```

### Troubleshooting

#### gpg failed to sign the data

If you get the following messages when trying to sign a commit or tag:

```
error: gpg failed to sign the data
error: unable to sign the tag
```

First, attempt to remove and re-insert the Yubikey. Then, make sure the card status lists correctly:

```
❯ gpg --card-status
```

If you see:

```
PIN retry counter : 0 0 3
```

This means you have blocked the normal PIN due to many incorrect attempts. The third PIN represents the retry counter for the Admin PIN.

Unblock the normal PIN by entering the Admin PIN:

```
❯ gpg --card-edit

gpg/card> admin
Admin commands are allowed

gpg/card> passwd
gpg: OpenPGP card no. … detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 2
PIN unblocked and new PIN set.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? q
```

## macOS integration

### Offline authentication using HMAC-SHA1 Challenge-Response

The new Yubikeys support a new authentication mechanism called HMAC-SHA1 Challenge-Response which enables offline authentication. This is very useful for working machines as internet access is not required for authentication.

The main use case is to require 2FA for escalating privileges to `root`. For example, the Yubikey can be configured to be required in addition to providing the root password.

#### Configuring HMAC-SHA1 Challenge-Response

The first step is to set up the Yubikey for HMAC-SHA1 Challenge-Response authentication. This can be done either with the [Yubikey Personalization Tool](https://itunes.apple.com/us/app/Yubikey-personalization-tool/id638161122?mt=12) or via the `ykpersonalize` command-line utility.

##### Using the _ykpersonalize_ command-line utility

First, start by installing `ykpers`:

```sh
❯ brew install ykpers
```

Then:

```sh
❯ ykpersonalize -2 -ochal-resp -ochal-hmac -ohmac-lt64 -ochal-btn-trig
```

Which basically means:

- Use slot 2 (`-2`)
- Set challenge-response mode (`-ochal-resp`)
- Generate HMAC-SHA1 challenge responses (`-ochal-hmac`)
- Calculate HMAC on less than 64 bytes input (`-ohmac-lt64`)
- The Yubikey will allow its serial number to be read using an API call (`-oserial-api-visible`). -

##### Using the _Yubikey Personalization Tool_

1. Plug in your Yubikey
2. Click _Challenge-Response_
3. Select _HMAC-SHA1_ mode
4. Choose _Configuration Slot 2_
5. Select _Require user input (button press)_
6. Select _Variable input_ as HMAC-SHA1 mode
7. Click _Write Configuration_ and don't save any logging file as it exposes the secret key written to the Yubikey

##### Generating the initial challenge

Install `pam_yubico`:

```sh
❯ brew install pam_yubico
❯ mkdir -m0700 -p ~/.yubico
```

Generate the initial challenge request:

```
❯ ykpamcfg -2
```

##### Enable Challenge-Response authentication module

Confirm the `pam_yubico.so` file exists to avoid being locked out of `sudo`:

```sh
❯ test -e /usr/local/opt/pam_yubico/lib/security/pam_yubico.so && echo "File exists, you may proceed."
```

Start a new shell session with `sudo` just to make sure you can still find your way out in case there is an error with the PAM file.

Then edit `/etc/pam.d/sudo` on another shell session and add the following line as the first one:

```sh
auth       required     /usr/local/opt/pam_yubico/lib/security/pam_yubico.so mode=challenge-response
```

Confirm you need to touch the Yubikey by running the following command on a new shell session:

```sh
❯ sudo -l
```

Notice that you will actually need to touch the Yubikey twice - one to verify the current challenge on file and another to generate a new challenge-response on success.

### Login and keychain authentication

macOS Sierra (10.12) ships with an improved SmartCard integration, including support for SmartCard Extensions

<sup>1</sup>

, an updated CCID driver and bug fixes on the PC/SC framework

<sup>2</sup>

. This translates into a clean and simple configuration for using a Yubikey to login to the macOS.

The login screen as well as Keychain-related operations will show a _PIN_ placeholder instead of _Password_ if the Yubikey is plugged in and has been paired before.

**References**

1. <https://developer.apple.com/library/content/releasenotes/MacOSX/WhatsNewInOSX/Articles/OSXv10.html>

2. <https://ludovicrousseau.blogspot.pt/2016/09/macos-sierra-and-smart-cards-status.html>

#### Managing pairing

The `sc_auth` utility (_smart card authorization setup script_) bundled by macOS can be used for a variety of SmartCard related operations.

1. List all SmartCard hashes for user `foobar`:

  ```sh
  ❯ sc_auth list -u foobar
  ```

2. Unpair a SmartCard hash `bar` from user `foobar`:

  ```sh
  ❯ sc_auth unpair -u foobar -h bar
  ```

3. Unpair all SmartCards from user `foobar`:

  ```sh
  ❯ sc_auth unpair -u foobar
  ```

4. Disable the SmartCard Pairing UI whenever a new SmartCard is entered:

  ```sh
  ❯ sc_auth pairing_ui -s disable
  ```
