#### Enabling touch protection

Install [https://developers.yubico.com/yubikey-manager/](Yubikey-manager) on a permanent destination:

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
