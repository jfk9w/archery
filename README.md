# archery

Personal Arch Linux package repository.

## Packages

- `amneziawg-linux-lts-bin` — AmneziaWG kernel module prebuilt for the exact
  `linux-lts` package version used during the build.

## Local build

Install `base-devel` and the current `linux-lts-headers`, then run:

```bash
./scripts/prepare-linux-lts packages/amneziawg-linux-lts-bin/PKGBUILD
cd packages/amneziawg-linux-lts-bin
makepkg --syncdeps --cleanbuild
```

`prepare-linux-lts` pins both the pacman package version and the kernel release.
Consequently, pacman will refuse to upgrade `linux-lts` until a matching module
package is available.

## Publishing

The build workflow creates a signed pacman repository and deploys it through
GitHub Pages. Before running it:

1. Enable GitHub Pages with **GitHub Actions** as its source.
2. Create a dedicated, preferably unencrypted repository signing key.
3. Export the private key, encode it with `base64 --wrap=0`, and save it as the
   `ARCHERY_GPG_PRIVATE_KEY` Actions secret.

The public key and its fingerprint are published as `archery.gpg` and
`archery.fingerprint`.

## Client setup

Download the repository key and inspect its fingerprint:

```bash
curl --fail --silent --show-error --location \
  https://jfk9w.github.io/archery/archery.gpg \
  --output /tmp/archery.gpg

gpg --show-keys --with-fingerprint /tmp/archery.gpg
```

Verify the displayed fingerprint against a trusted copy provided by the
repository owner. Then import the key into pacman's keyring and locally sign it:

```bash
sudo pacman-key --add /tmp/archery.gpg
sudo pacman-key --lsign-key FINGERPRINT
```

Add the repository before the official repositories in `/etc/pacman.conf`:

```ini
[archery]
SigLevel = Required DatabaseOptional
Server = https://jfk9w.github.io/archery/$arch
```

Refresh the databases and install the package in the same transaction, so the
kernel and its module cannot get out of sync:

```bash
sudo pacman -Syu amneziawg-linux-lts-bin
```

Kernel-module tags are not automatically followed: their version number does
not reliably identify protocol compatibility with the separately released
userspace tools. Run the update workflow manually with an explicitly tested
version. It updates the pinned version and checksum and commits them to `main`;
that commit triggers a package build.
