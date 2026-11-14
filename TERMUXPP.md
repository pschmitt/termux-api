# TermuxPP:API

This fork targets `dev.pschmitt.termux` and installs as
`dev.pschmitt.termux.api`. The API Java namespace is also
`dev.pschmitt.termux.api`; shared library classes retain `com.termux.shared`.

## Build

Clone with `git clone --recurse-submodules`, or run
`git submodule update --init --recursive` after cloning. The `termux-app`
submodule pins the TermuxPP shared libraries; do not replace them with the
upstream Maven artifact, whose paths and package identities target stock Termux.

With JDK 17, Android SDK 35, and the NDK version in `gradle.properties`:

```sh
./gradlew :app:assembleDebug :app:lintDebug
```

Run substantial builds on a remote build worker, not a workstation.

## Install and verify

All TermuxPP apps sharing `dev.pschmitt.termux` must use the same signing
certificate. The checked-in upstream test key is identical to the core's test
key and is **public and untrusted**, suitable only for development. Production
builds must use the same private signing key as the installed TermuxPP core.

Install alongside stock Termux:API, not over it. Install the command-line tools
from <https://github.com/pschmitt/termux-api-package>; the stock `termux-api`
package targets the upstream app and is not interchangeable with this fork.
Grant Android permissions only as needed. Test `termux-battery-status`, clipboard
access, and a notification, including after force-stopping and reopening the app.
Verify that stock Termux and its API continue to operate independently.

The stock `termux-am` package is another compatibility dependency: its embedded
`FakeContext` identifies the caller as `com.termux`. Use the matching
[TermuxAm fork](https://github.com/pschmitt/TermuxAm), whose default caller is
`dev.pschmitt.termux`. Filesystem path rewriting alone cannot fix this identity.

The core repository's `scripts/package-helpers.py` packages both helpers as
normal `termux-am` and `termux-api` debs. Install the device-mode packages inside
TermuxPP, then run `apt-mark hold termux-am termux-api`. This satisfies nixpp's
APT package requirements without allowing stock upgrades to replace the fork
helpers. See the core's `docs/en/aarch64-bootstrap.md` for build and installation.

On-device smoke tests cover battery status with the API app already running,
battery status after force-stopping it (broadcast fallback), and toast delivery.

No permissions or shared-UID authorization checks have been relaxed.
