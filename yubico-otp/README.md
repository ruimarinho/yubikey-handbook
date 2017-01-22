## 2FA via Yubico OTP (server)

Improving the security of the client (authenticating SSH with PIV) and simplifying the managing tasks on the server (authenticating SSH via User Certificates) is the first step for a more secure environment. However, they still rely on a single factor for authentication. This can be changed to require a second factor (2FA).

Enabling 2FA with SSH or `sudo` on a remote server requires online validation, either by using YubiCloud's OTP Validation service backed by their own YubiHSM (Hardware Security Module), or hosting it elsewhere inside a private network.

The main benefit of running the OTP validation service on a private server is that the AES keys programmed into the Yubikeys are in complete control. YubiCloud, on the other hand, manages those AES keys for us, so it has the benefit of providing a seamless, no-maintenance experience.

YubiCloud offers a premium service with statistics (personalized monthly report via email, server uptime and Yubikey usage reports), including an SLA for an undisclosed price.

Other means of offline validation, such as HMAC-SHA1 Challenge-Response or U2F, are not supported at the moment due to architectural constraints (HMAC-SHA1 Challenge-Response would require the Yubikey to be plugged-in on the remote server) or lack of software support ([OpenSSH does not support the U2F standard yet](https://bugzilla.mindrot.org/show_bug.cgi?id=2319)).
