# archery

Personal Arch Linux package repository.

## Packages

### `amneziawg-linux-lts`

A prebuilt AmneziaWG kernel module for Arch Linux's `linux-lts` kernel. Each
package is built for one exact kernel package version and depends on that same
version of `linux-lts`. This prevents pacman from upgrading the kernel before a
matching module package has been published.

The package installs the compressed module under
`/usr/lib/modules/<kernel>/updates/archery` and provides `AMNEZIAWG-MODULE` and
`amneziawg-dkms`. It conflicts with the DKMS, Git DKMS and other prebuilt
AmneziaWG module variants, including the former `amneziawg-linux-lts-bin` name.
It contains only the kernel module; install `amneziawg-tools` separately for the
userspace commands and configuration helpers.

Its version has the form `<module-version>.<kernel-package-version>-1`, for
example `1.0.20260329.6.18.44.1-1`.

### `caddy`

A custom, statically linked Caddy binary built with `xcaddy`. It includes:

- Porkbun DNS support (`github.com/caddy-dns/porkbun`);
- Selectel DNS support (`github.com/jfk9w-go/caddy-dns-selectel`);
- layer 4 proxying (`github.com/mholt/caddy-l4`);
- response body replacement (`github.com/caddyserver/replace-response`).

The package has the same name as the official Arch package and the `archery`
repository is placed before the official repositories, so this build takes
precedence. Like the official package, it creates the `caddy` system user and
runtime/data directories, installs Caddyfile and API systemd units, provides a
default `/etc/caddy/Caddyfile` and serves a welcome page from
`/usr/share/caddy`. The default Caddyfile is tracked as a pacman backup file, so
local configuration is preserved across package upgrades. Shell completions are
installed for bash, fish and zsh. The `mailcap` dependency supplies the system
MIME type database used by Caddy's file server. Runtime state, including ACME
accounts, certificates and the autosaved configuration, is stored below
`/var/lib/caddy`.

Its version has the form `<caddy-version>.<package-change-timestamp>-1`. The UTC
timestamp comes from the latest commit that changed `packages/caddy` or its
preparation script. Consequently, changing a plugin produces an upgrade even
when the upstream Caddy version stays the same.

## Local build

Install `base-devel` and the current `linux-lts-headers`, then run:

```bash
./scripts/prepare-linux-lts packages/amneziawg-linux-lts/PKGBUILD
cd packages/amneziawg-linux-lts
makepkg --syncdeps --cleanbuild
```

For Caddy:

```bash
./scripts/prepare-caddy packages/caddy/PKGBUILD
cd packages/caddy
makepkg --syncdeps --cleanbuild
```

`prepare-linux-lts` pins both the pacman package version and the kernel release.
Consequently, pacman will refuse to upgrade `linux-lts` until a matching module
package is available.

The scheduled Caddy update workflow follows the latest non-prerelease GitHub
release automatically. Patch updates are committed directly to `main` seven
days after publication, and minor updates after fourteen days. Major updates
open a pull request after thirty days and require a manual merge. Direct commits
dispatch the normal package build and publication workflow; merging a major
update pull request triggers it through the push to `main`.

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
sudo pacman -Syu amneziawg-linux-lts
```

Kernel-module tags are not automatically followed: their version number does
not reliably identify protocol compatibility with the separately released
userspace tools. Run the update workflow manually with an explicitly tested
version. It updates the pinned version and checksum and commits them to `main`;
that commit triggers a package build.
