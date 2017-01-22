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
