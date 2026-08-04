# M0 — Environment

Fill this in before starting M1. Everything downstream depends on these being pinned.

## Pinned versions

| Thing | Value |
|---|---|
| LFS book version | _e.g. 12.2 — **stable release, not `development`**_ |
| Init system | _sysvinit \| systemd — the book is a different book for each_ |
| Book URL | |
| Target architecture | _x86_64_ |
| Host distro + version | |
| Host kernel | _output of `uname -r`_ |

Mixing book versions across chapters is the single most common cause of unexplainable LFS
failures. Pick one, record it here, and never follow a different one.

## Host

| | |
|---|---|
| Hypervisor | _QEMU/KVM \| VirtualBox_ |
| KVM available | _`ls -l /dev/kvm` — if absent, builds run under emulation and take days_ |
| vCPU / RAM / disk | |
| `MAKEFLAGS` | _e.g. `-j4` — match vCPU count_ |

## Disks

| Purpose | Device | Size | Filesystem | UUID |
|---|---|---|---|---|
| LFS root | | | ext4 | |
| swap | | | swap | |
| ESP (if UEFI) | | | vfat | |

Always mount and reference by **UUID**. Device names reorder between boots and that failure
is deliberately reproduced in M7.

## Snapshot policy

- Snapshot before every chapter: `lfs-ch<N>-pre`
- Keep the last 3, plus a permanent `lfs-m6-first-boot` once the system boots
- The M6 VM is **kept** through M9 — it's the reference to diff a diverging rebuild against

| Snapshot | Taken | Purpose |
|---|---|---|
| | | |

## Host requirements check

```
$ ./scripts/host-check.sh
```

Paste the output here. Every failure must be fixed before M1, not worked around.

## Notes
