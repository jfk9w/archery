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
`archery.fingerprint`. On a client, import and locally sign that key, then add:

```ini
[archery]
SigLevel = Required DatabaseOptional
Server = https://jfk9w.github.io/archery/$arch
```

The update workflow checks the latest upstream AmneziaWG tag daily. When it
changes, the workflow updates the pinned version and checksum and commits them
to `main`; that commit triggers a package build.
