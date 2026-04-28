# CLAUDE.md

## Project purpose
VyOS fork of Debian's [`live-build`](https://wiki.debian.org/DebianLive) — the toolchain that assembles a hybrid live ISO from a Debian package set. Consumed by `vyos/vyos-build` to produce the VyOS ISO; carries VyOS-specific patches on top of Debian's `live-build` upstream.

## Tech stack
- Shell-heavy. Standard Debian `live-build` codebase: shell scripts, Perl helpers, `data/` skeleton trees, `share/` recipes, `manpages/`, `frontend/`, `examples/`.
- Debian packaging in `debian/`. Build-depends (`debian/control`): `debhelper-compat (= 12)`, `po4a`, `gettext`.

## Build / test / run
```
dpkg-buildpackage -uc -us -tc -b      # build the live-build .deb
# Direct toolchain invocation usually goes through vyos/vyos-build's docker image,
# which installs this package and calls `lb config`/`lb build`.
```
No standalone test suite — exercised end-to-end by `vyos-build`'s ISO assembly.

## Repository layout
- `frontend/` — `lb` user-facing CLI front-end.
- `scripts/` — stage scripts (config, bootstrap, chroot, binary, source).
- `share/` — recipe data (package lists, hooks, includes).
- `data/` — base data files (e.g. flavors, locales).
- `functions/` — shell functions library.
- `manpages/` — man pages (po4a-translated).
- `examples/` — example configs.
- `debian/` — Debian packaging.
- `Makefile` — top-level dispatcher.
- `COPYING` — license.

## Cross-repo context
- Consumed by `vyos/vyos-build` at ISO assembly time. `vyos-build` invokes `lb config`/`lb build` inside its build container after dropping the `vyos-live-build` `.deb` in.
- Listed indirectly in the VyOS image build chain. Not in `VyOS-Networks/vyos-build-packages/repos.toml`'s 14-package list (which is the Debian source-package set baked into VyOS itself), but is a build-time dependency of the ISO toolchain.

## Conventions
- Commit/PR title: `component: T12345: description` (Phorge task ID at https://vyos.dev) where applicable.
- License: GPLv3 (inherited from Debian `live-build`).
- Maintain the upstream vendor field in `debian/control` (`Maintainer: Debian Live <debian-live@lists.debian.org>` + VyOS uploaders) — keeps the Debian heritage transparent.
- Default branch: confirm via `git ls-remote --symref`; LTS-train branches may carry train-specific patches.

## Mirror relationship
Mirror twin: `VyOS-Networks/vyos-live-build`. Mirror pipeline not confirmed live for this repo. Treat `vyos/*` as canonical.

## Notes for future contributors
- This is a vendor fork. **Keep VyOS-specific deltas minimal and clearly attributed** so periodic resync with Debian upstream remains tractable.
- ISO behavior changes that look like `live-build` bugs are often package-set issues handled by `vyos-build`; check there first before patching here.
- No GitHub Actions workflows are currently configured in this repo (at the time of the most recent shallow clone), so CI verification happens implicitly via `vyos-build`'s smoketest.
- When bumping `live-build` from upstream Debian, expect cascading regressions in `vyos-build` ISO assembly — coordinate.

---

This file is mirrored on Confluence: [`vyos/vyos-live-build`](https://internal.confluence.vyos.com/wiki/spaces/VYOS/pages/818544949). The Confluence page also carries the per-repo audit data (settings, workflows, secret counts, hygiene) that complements this CLAUDE.md. Edit either side; resync via the documentation pipeline.
