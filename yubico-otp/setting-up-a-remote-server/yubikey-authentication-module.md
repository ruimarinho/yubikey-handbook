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
