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

