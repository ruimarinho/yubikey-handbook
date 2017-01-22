#### Installing libpam-yubico

The package `libpam-yubico` provides the requires libraries to integrate Yubico's software into PAM (Linux Pluggable Authentication Modules).

Add the PPA and install the package:

```sh
❯ apt-get install -y vim software-properties-common
❯ add-apt-repository -y ppa:yubico/stable
❯ apt-get update
❯ apt-get install -y libpam-yubico
```
