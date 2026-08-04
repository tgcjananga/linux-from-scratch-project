# scripts/ — verification and build automation

Collectively, these are **the package manager LFS doesn't give me**. Writing them by hand is
what makes Portage/apt legible later: each script corresponds to something a real package
manager does for you.

## Conventions

- `#!/usr/bin/env bash` + `set -euo pipefail` in every script
- No script assumes it runs from a particular directory
- Exit 0 = pass, non-zero = fail, so they compose and can gate a milestone
- Every script prints what it checked, not just a verdict

## Planned

| Script | Milestone | Purpose | Distro equivalent |
|---|---|---|---|
| `host-check.sh` | M0 | LFS host requirements + my additions (disk, `$LFS`, user) | build-dep check |
| `disk-report.sh` | M1 | partitions, UUIDs, filesystems, mounts; flag non-UUID fstab entries | — |
| `inspect-binary.sh` | M3 | full ELF story for any binary (`file`/`ldd`/`readelf`/`nm`) | — |
| `build-pkg.sh` | M5 | configure/make/install with logging, timing, exit status | ebuild / dpkg-buildpackage |
| `file-manifest.sh` | M5 | filesystem diff before/after install → installed-file list | Portage's file DB / `dpkg -L` |
| `first-boot-verify.sh` | M6 | kernel version, mounts, network, services, `dmesg` errors | — |

`first-boot-verify.sh` is also the M9 acceptance test: the rebuilt system must pass the same
script, unmodified. Don't weaken it to make the rebuild pass — fix the rebuild.
