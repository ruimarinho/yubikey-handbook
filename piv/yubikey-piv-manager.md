### Yubikey PIV Manager

The [Yubikey PIV Manager](https://developers.yubico.com/yubikey-piv-manager) is a GUI tool to manage PIV-related data on a Yubikey. The command-line version can be installed via Homebrew:

```
❯ brew install yubico-piv-tool
```

The command-line utility is more powerful (e.g. allows overriding pin and touch default slot policies) but it also more prone to user errors. It's very easy to overwrite an existing key, for instance.

After [downloading](https://developers.yubico.com/yubikey-piv-manager) and installing Yubikey PIV Manager and `yubico-piv-tool`, insert your Yubikey. You will be prompted [initialize your device](../device-initialization/README.md).
