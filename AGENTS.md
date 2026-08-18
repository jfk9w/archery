# Repository guidance

This repository builds and publishes a personal signed Arch Linux package
repository for x86_64 systems. Keep changes small, reproducible and compatible
with unattended GitHub Actions builds.

## Package rules

- Keep package-specific files under `packages/<pkgname>` and reusable build or
  update logic under `scripts`.
- Preserve `Igor Kulkov <arch.i@sonc.top>` as the package maintainer.
- Pin downloaded sources and verify them with SHA-256 checksums. Do not replace
  checksums with `SKIP`.
- Build packages as the unprivileged `builder` user in CI.
- Run `bash -n` for changed shell scripts and `git diff --check` before handing
  off changes.

### AmneziaWG

- `amneziawg-linux-lts` must be built against the exact `linux-lts` and
  `linux-lts-headers` package version installed in the build environment.
- Keep `_kernel_pkgver`, `_kernel_release`, the exact `linux-lts` dependency and
  the installed module directory in sync.
- Do not automatically follow AmneziaWG module tags. Module updates are selected
  manually through the `Update AmneziaWG` workflow after compatibility testing.
- Preserve conflicts with DKMS and alternative prebuilt module packages.

### Caddy

- Build Caddy through `xcaddy`; keep every non-upstream module explicitly pinned
  to a tag or commit.
- When changing the module set, update the module assertions in `check()` and the
  package description in `README.md` together.
- Do not derive `_build_timestamp` from workflow run time. `prepare-caddy` uses
  the latest relevant Git commit so identical sources retain an identical
  package version.
- The scheduled updater follows only GitHub's latest non-prerelease Caddy
  release. Patch releases wait 7 days, minor releases wait 14 days, and major
  releases open a manually merged pull request after 30 days.
- Manual Caddy or plugin edits build immediately with the `_caddy_version`
  recorded in `PKGBUILD`; they do not bypass or modify the scheduled updater's
  release policy.

## Publishing

- `build.yml` builds all packages, signs packages and repository databases, and
  deploys the result to GitHub Pages.
- The repository signing private key is supplied only through the
  `ARCHERY_GPG_PRIVATE_KEY` Actions secret. Never add private key material to the
  repository or logs.
- Keep the published repository name `archery` and retain regular copies of the
  database links because GitHub Pages artifacts do not support symbolic links.
