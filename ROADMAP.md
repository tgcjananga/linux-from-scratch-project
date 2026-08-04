# Linux From Scratch — Engineering Roadmap

## Goal

Build a bootable Linux system entirely from source, and come out of it able to explain
and reproduce every layer: toolchain, kernel, init, boot chain, and userspace. The end
state is not "LFS booted once" — it is **a rebuild I can do from my own notes**, which is
the prerequisite for treating Gentoo as a set of automated decisions rather than magic.

## How this roadmap works

Work runs in **two tracks that stay coupled**:

| Track | Output | Rule |
|---|---|---|
| **Theory** | `docs/`, `diagrams/` | Nothing is "learned" until it is written down in my own words |
| **Implementation** | `experiments/`, `journal/`, `scripts/`, `configs/` | Nothing is "understood" until I've run it, broken it, and fixed it |

Theory leads implementation by **one milestone**. Read about the boot chain before you
have to debug it at 1 AM; read about the toolchain before Chapter 5 fails.

Each milestone has an **Exit Criteria** section. These are deliberately testable — a
command that exits 0, a file that exists, or a question I can answer with the notes
closed. A milestone is not done because the reading is done. It is done when the exit
criteria pass.

**Estimates** are calendar guesses at ~8–10 focused hours/week. They will be wrong.
Track actuals in `journal/` and adjust; the ordering matters far more than the dates.

---

## Milestone map

```
M0  Environment            ── reproducible build VM + pinned book version
M1  Systems literacy       ── FHS, storage, filesystems  (+ verify scripts)
M2  Boot chain             ── theory + annotated trace of a REAL boot
M3  Program → process      ── ELF, processes, memory     (running experiments)
M4  Toolchain              ── cross-compilation proven, not just read
M5  LFS to chroot          ── Ch. 5–7: temp toolchain, entered chroot
M6  Bootable system        ── Ch. 8–11: kernel, GRUB, first login
M7  Break / fix            ── recovery runbook from real self-inflicted failures
M8  Internals lab          ── syscall + kernel-module experiments on my own system
M9  Reproducible rebuild   ── rebuild from my notes, in a second VM, timed
M10 Reflection & handoff   ── what LFS taught; what Gentoo automates
```

Critical path: **M0 → M4 → M5 → M6**. M1–M3 and M8 are broad-but-shallow-able; if time
gets tight, thin those, never M4. A misunderstood toolchain makes M5 unrecoverable.

---

# M0 — Environment

**Objective:** a build environment I can destroy and recreate without losing work.

### Tasks

- [ ] Repo initialized, pushed, branch strategy decided (`develop` for notes, tag milestones)
- [ ] Host: QEMU/KVM installed, verify `kvm-ok` / `/dev/kvm` present (KVM, not TCG — LFS
      builds are CPU-bound and emulation costs you days)
- [ ] Build VM: Debian stable or Ubuntu LTS, **≥4 vCPU, ≥8 GB RAM, ≥60 GB disk**
- [ ] Run the LFS `version-check.sh` from the book's host requirements chapter; fix every
      failure before continuing
- [ ] **Pin the book**: record exact LFS version + init system (sysvinit vs systemd) in
      `journal/00-environment.md`. Never mix versions across chapters.
- [ ] Snapshot policy written down: snapshot before every chapter, name them
      `lfs-ch<N>-pre`, keep the last 3
- [ ] `scripts/host-check.sh` — wraps version-check plus my own additions (disk free,
      `$LFS` set, correct user)

### Exit criteria

- `scripts/host-check.sh` exits 0 on a fresh clone of the VM
- A snapshot can be rolled back and the build environment still passes host-check
- `journal/00-environment.md` states the pinned book version and init system

**Artifacts:** `journal/00-environment.md`, `scripts/host-check.sh`
**Estimate:** 1 week

---

# M1 — Systems literacy

**Objective:** know the shape of a Linux system before assembling one — what lives where,
and what a disk looks like underneath a filesystem.

### Theory

- Kernel vs. userspace; syscalls as the boundary
- Shell, coreutils, and what "GNU/Linux" actually names
- FHS: `/bin` vs `/usr/bin`, the `/usr` merge, why `/lib64` exists, what `/run` is for
- Distro anatomy: what a distro *is* (kernel + libc + toolchain + package manager + policy)
- Package managers as a concept: dependency resolution, file ownership, why LFS has none

### Implementation

- Disks and partitioning: MBR vs GPT, UEFI vs BIOS boot, the ESP and why it's FAT32
- ext4 (journaling, inodes, extents), swap, the VFS layer
- Practice: `lsblk`, `blkid`, `findmnt`, `fdisk`/`gdisk`, `mkfs.ext4`, `mount -o`, `/etc/fstab`
- Build the LFS partition + swap on a second virtual disk; mount by **UUID**, not `/dev/sdX`

### Exit criteria

- `scripts/disk-report.sh` prints the LFS partition table, UUIDs, filesystem types, and
  mount points, and flags any `fstab` entry referencing a device node instead of a UUID
- I can explain, notes closed, why the ESP is FAT32 and what breaks if `/boot` isn't readable
  by the bootloader
- `docs/filesystem-hierarchy.md` maps every top-level directory to *who writes it*
  (kernel, package install, admin, runtime)

**Artifacts:** `docs/linux-architecture.md`, `docs/filesystem-hierarchy.md`,
`docs/linux-distributions.md`, `docs/storage.md`, `docs/filesystems.md`,
`scripts/disk-report.sh`
**Estimate:** 2 weeks

---

# M2 — Boot chain

**Objective:** account for every stage from power-on to a shell prompt — and prove it
against a real boot, not a diagram.

### Theory

```
Power  →  UEFI firmware  →  ESP / GRUB (EFI binary)  →  GRUB config + modules
       →  kernel (vmlinuz, decompress, init early)   →  initramfs (find + mount root)
       →  switch_root  →  PID 1 (systemd/sysvinit)   →  targets/runlevels  →  getty → login
```

For each stage answer: *what code is executing, what does it need to find, where does it
find it, and what is the single most common failure?*

### Implementation

The tangible part — a **trace of the running VM**, matched stage by stage against the theory:

- [ ] `efibootmgr -v`, contents of the ESP, the GRUB EFI binary and `grub.cfg`
- [ ] `cat /proc/cmdline` — explain every parameter present
- [ ] Unpack the host's initramfs (`lsinitramfs` / `unmkinitramfs`) and inventory what's inside
- [ ] `dmesg` early lines + `systemd-analyze critical-chain` (or sysvinit script order)
- [ ] `diagrams/linux-boot.drawio` drawn from this evidence, annotated with the real paths
      and UUIDs from *this* machine

### Exit criteria

- Every parameter in `/proc/cmdline` is explained in `docs/linux-boot-process.md`
- The diagram names the actual artifact at each stage (path on disk, not a generic label)
- I can state which stage owns each of: "no bootable device", "GRUB rescue prompt",
  "kernel panic — not syncing: VFS unable to mount root fs", "emergency shell"

**Artifacts:** `docs/linux-boot-process.md`, `diagrams/linux-boot.drawio`
**Estimate:** 1.5 weeks

---

# M3 — Program → process

**Objective:** follow one C file all the way to an address in a running process's memory.

### Theory

```
C source → preprocessor → compiler → assembler → object (.o) → linker → ELF
         → loader (ld.so) → mapped segments → running process
```

- ELF: sections vs segments, symbol tables, relocations, PLT/GOT, dynamic vs static
- Processes: `fork`, `exec`, `wait`, PIDs, exit status, zombies/orphans, signals, scheduling
- Memory: virtual address space, paging, stack vs heap vs mmap, shared libraries, `brk`/`mmap`

### Implementation

`experiments/` — each with a `Makefile`, a `README.md` stating the question it answers, and
the observed output committed:

- [ ] `experiments/elf/` — same program static vs dynamic; compare with `file`, `ldd`,
      `readelf -h -l -S`, `nm`, `objdump -d`, `strings`. **Explain the size difference.**
- [ ] `experiments/elf/` — break it on purpose: strip symbols, corrupt the ELF magic,
      remove a needed `.so`; record the exact error each produces
- [ ] `experiments/processes/` — `fork`+`exec`+`wait`; create a zombie and an orphan and
      observe both in `ps`; catch `SIGTERM`, show `SIGKILL` can't be caught
- [ ] `experiments/memory/` — print `/proc/self/maps`, walk each region; grow the heap via
      `malloc` and watch it move; `mmap` a file and modify it through the mapping
- [ ] `scripts/inspect-binary.sh` — one command that dumps the ELF story for any binary

### Exit criteria

- `make -C experiments/elf && make -C experiments/processes && make -C experiments/memory`
  all succeed and each README records actual observed output
- I can point at a line in `/proc/self/maps` and say which build step put it there
- `docs/elf-format.md` explains why a dynamically linked `hello` is smaller but needs more
  at runtime — the intuition M4 depends on

**Artifacts:** `experiments/{elf,processes,memory}/`, `docs/elf-format.md`,
`docs/process-management.md`, `docs/memory-management.md`, `scripts/inspect-binary.sh`
**Estimate:** 3 weeks

---

# M4 — Toolchain

**Objective:** understand *why LFS builds a compiler twice* before doing it. This is the
milestone that decides whether M5 is a build or a mystery.

### Theory

- binutils (`as`, `ld`) → gcc → glibc → and back to gcc: the circular dependency, and how
  a temporary toolchain breaks the circle
- Host contamination: how a compiler silently picks up host headers/libs, and why that
  poisons a from-scratch system
- Target triplets (`x86_64-pc-linux-gnu` vs `x86_64-lfs-linux-gnu`) — why LFS invents one
- `--prefix`, `--with-sysroot`, `DESTDIR`, the dynamic loader path, `specs` files
- Static vs dynamic linking of the temporary tools, and *why pass 1 differs from pass 2*

### Implementation

Don't take it on faith — prove it small before proving it large:

- [ ] Cross-compile a `hello.c` with an existing cross toolchain; confirm with
      `readelf -h` that the target arch differs from the host
- [ ] On a host binary, dump the search paths: `gcc -print-search-dirs`,
      `gcc -dumpspecs`, `ld --verbose | grep SEARCH_DIR`
- [ ] Demonstrate contamination deliberately: link against a host lib that shouldn't be
      visible, and record what the failure looks like — this is the error you'll hit in Ch. 5

### Exit criteria

- `docs/toolchain.md` answers, in my own words: why two passes, why a separate triplet,
  what `--with-sysroot` changes, and what would break if I skipped the temp toolchain
- I can predict what `gcc -print-search-dirs` *should* look like inside chroot before I get there
- A cross-compiled binary exists in `experiments/` with the `readelf` output proving its arch

**Artifacts:** `docs/toolchain.md`, `experiments/toolchain/`
**Estimate:** 2 weeks

---

# M5 — LFS to chroot

**Objective:** temporary toolchain built, chroot entered, and every package accounted for.

Work the book chapter by chapter (roughly Ch. 4–7). **Snapshot before each chapter.**

### Per-package discipline

For every package, `journal/` records four answers — not copied from the book:

1. **What is it?** one sentence
2. **Why is it needed here, at this point in the order?**
3. **What did it install?** — captured mechanically, not guessed (see below)
4. **How do I verify it?** — a command and its expected output

### Implementation

- [ ] `scripts/build-pkg.sh` — wraps each build: `set -euo pipefail`, tees full output to
      `logs/<pkg>.log`, times it, records the exit status. Build logs are the primary
      evidence when something breaks three chapters later.
- [ ] `scripts/file-manifest.sh` — snapshot the filesystem before/after a package and diff,
      so "which files are installed" is *measured*. This is package management from first
      principles, and directly foreshadows what Portage does for you.
- [ ] Chroot understood as mechanism, not incantation: virtual kernel filesystems
      (`/dev`, `/proc`, `/sys`, `/run`) and *why each is bind-mounted*; `chroot` vs
      `pivot_root`; why the environment is scrubbed (`env -i`)
- [ ] Build a minimal chroot by hand — busybox + a shell — separately from LFS, to see the
      mechanism with nothing else in the way

### Exit criteria

- `chroot` entered successfully with the documented environment
- One journal entry per package, all four questions answered
- `logs/` contains a complete log for every package built
- `docs/chroot.md` explains what would break if `/proc` weren't mounted, and I can
  reproduce that failure on the hand-built chroot

**Artifacts:** `journal/ch04..ch07-*.md`, `docs/chroot.md`, `scripts/build-pkg.sh`,
`scripts/file-manifest.sh`, `logs/`
**Estimate:** 4 weeks — the longest stretch, mostly compile time

---

# M6 — Bootable system

**Objective:** LFS boots to a login prompt on hardware I control.

### Implementation

- [ ] Final system packages (Ch. 8) — same per-package discipline as M5
- [ ] System configuration (Ch. 9): network, `/etc/fstab` (UUIDs), locale, `/etc/hosts`,
      console, init configuration
- [ ] Kernel (Ch. 10): navigate the source tree, `make menuconfig` from a known baseline,
      and **justify every option I change** — filesystem support built in vs. modular,
      the drivers this VM's virtual hardware needs, `CONFIG_` options the book requires
- [ ] Decide initramfs or not, **and write down why**. LFS can boot without one if the root
      filesystem driver and disk controller are built in — understanding *that* trade-off
      is the point.
- [ ] GRUB (Ch. 11) installed manually: EFI binary to the ESP, `grub.cfg` written by hand,
      not generated. Understand `root=`, `set root`, `linux`/`initrd` lines.
- [ ] First boot; verify login, `uname -a`, networking, mounts, running services

### Exit criteria

- Cold boot to login prompt, no host kernel and no host initramfs involved
- `scripts/first-boot-verify.sh` runs inside LFS and checks: kernel version matches the
  one built, all `fstab` entries mounted, network route present, expected services up, and
  no `dmesg` errors above a documented allowlist — exits 0
- `configs/kernel.config` committed with `docs/kernel.md` explaining the non-default choices
- `grub.cfg` committed and every line explained in `docs/grub.md`

**Artifacts:** `journal/ch08..ch11-*.md`, `configs/kernel.config`, `configs/grub.cfg`,
`docs/kernel.md`, `docs/grub.md`, `scripts/first-boot-verify.sh`
**Estimate:** 4 weeks
**Tag the repo here.** This is the halfway point in effort and 100% of the "it boots" reward.

---

# M7 — Break / fix

**Objective:** a recovery runbook written from failures I actually caused and fixed. This is
the milestone that converts a working system into transferable skill.

Snapshot first. Then break exactly one thing at a time, and for each record: **symptom →
what stage failed → how I diagnosed it → the fix → how to avoid it.**

| # | Break | Expected failure stage |
|---|---|---|
| 1 | Wrong UUID in `/etc/fstab` | initramfs / early mount |
| 2 | Remove root filesystem support from the kernel | kernel: cannot mount root |
| 3 | Corrupt `grub.cfg` | GRUB rescue prompt |
| 4 | Delete the kernel image from `/boot` | GRUB: file not found |
| 5 | Break the dynamic loader / `ld.so.conf` | PID 1 fails, everything "not found" |
| 6 | Wrong init path / bad `init=` | kernel panic, no PID 1 |
| 7 | Full root filesystem | services fail in confusing ways |

At least #1–#5 must be recovered **without reverting the snapshot** — that's the whole
exercise. Fix from a rescue shell, a live ISO, or GRUB's command line.

### Exit criteria

- One file per failure in `troubleshooting/`, each with a real transcript of the recovery
- `troubleshooting/README.md` is a symptom-indexed table: *"I see X → check Y"*
- At least one failure recovered entirely from the GRUB command line

**Artifacts:** `troubleshooting/`
**Estimate:** 2 weeks

---

# M8 — Internals lab

**Objective:** use the system I built as a laboratory — including kernel-side, which is
only really available to me now that I control the kernel config.

- [ ] `experiments/process/` — pipes and IPC: `pipe()` + `fork()`, shared memory, and the
      shell pipeline reimplemented (`a | b` from scratch)
- [ ] `experiments/network/` — TCP socket server/client; watch it with `ss`, `strace`,
      `tcpdump`; trace a connection through `/proc/net/`
- [ ] `experiments/kernel/` — an out-of-tree "hello world" module: build against my own
      kernel tree, `insmod`/`rmmod`, `dmesg`; then a module exposing a `/proc` or sysfs
      entry read from userspace. **Building this against a kernel I compiled myself is the
      payoff for M6** — no distro headers package involved.
- [ ] `strace` a trivial program end to end and explain every syscall before `main`

### Exit criteria

- Each experiment directory builds and runs on the LFS system, output committed
- The kernel module loads on the LFS kernel and its `dmesg` output is captured
- `docs/syscall-boundary.md` traces one syscall from the C call to kernel entry

**Artifacts:** `experiments/{process,network,kernel}/`, `docs/syscall-boundary.md`
**Estimate:** 3 weeks

---

# M9 — Reproducible rebuild

**Objective:** prove the knowledge is mine and the documentation is real.

Rebuild the whole system **in a second, fresh VM**, primarily from my own notes. Keep the
M6 VM — when the rebuild diverges, the original is the only reference for what "correct"
looked like. (Deleting it would remove the ability to diff.)

Rules:

- Notes and scripts first; open the book only when blocked
- **Log every book consultation** in `journal/rebuild.md` with what my notes were missing
- Fix the gap in the docs immediately — those fixes are the real output of this milestone
- Time it, and compare against the M5+M6 actuals

### Exit criteria

- Second system boots and passes `scripts/first-boot-verify.sh`
- `journal/rebuild.md` lists every book lookup and the doc commit that closed each gap
- Rebuild wall-clock recorded next to the original

**Artifacts:** `journal/rebuild.md`, doc corrections across `docs/`
**Estimate:** 2 weeks

---

# M10 — Reflection & handoff

**Objective:** consolidate, and set up Gentoo deliberately rather than by momentum.

`docs/reflection.md`:

- What surprised me
- What clicked, and what is still fuzzy (be specific — this becomes the Gentoo reading list)
- LFS vs. Ubuntu: what a distro adds on top of what I built
- **What Gentoo automates** — mapped explicitly to milestones: `build-pkg.sh` → ebuilds,
  `file-manifest.sh` → Portage's file database, kernel config → `genkernel` / manual,
  the toolchain dance → `crossdev` and stage3 tarballs, USE flags → the `./configure`
  choices I made by hand
- Where LFS is a poor model for daily use, and why (no package manager, no security updates)

`README.md`: final structure, how to navigate the repo, tags per milestone.

### Exit criteria

- `docs/reflection.md` contains the LFS→Gentoo mapping table
- README indexes every doc, experiment, and script
- Repo tagged, and the Gentoo project scoped in one page (`docs/next-gentoo.md`)

**Estimate:** 1 week

---

## Repository layout

```
ROADMAP.md              this file
README.md               index + navigation
docs/                   theory track: concepts in my own words
diagrams/               boot chain, address space, toolchain flow
experiments/            implementation track: small programs that answer one question each
  elf/ processes/ memory/ toolchain/ process/ network/ kernel/
journal/                build log, chapter by chapter, plus the rebuild diary
configs/                kernel.config, grub.cfg, fstab — the artifacts that define the system
scripts/                verification and build automation (the "package manager I don't have")
logs/                   raw build output per package (evidence for later debugging)
troubleshooting/        symptom-indexed recovery runbook
```

## Standing rules

1. **Snapshot before every chapter.** Non-negotiable. The cost of not doing it is days.
2. **Never copy the book into `docs/`.** If I can't write it in my own words, I haven't
   learned it. `docs/` is understanding; the book is reference.
3. **Every claim in a doc is verifiable by a command**, and that command is in the doc.
4. **Commit per package/experiment**, not per session — the git log becomes a build timeline.
5. **Log failures, including embarrassing ones.** The failure transcript is worth more than
   the success; M7 and M9 are built entirely out of them.
6. **Pin versions.** Book version, kernel version, package versions. Reproducibility is the
   whole point of M9.

## Progress

| Milestone | Status | Started | Finished | Notes |
|---|---|---|---|---|
| M0 Environment | ☐ | | | |
| M1 Systems literacy | ☐ | | | |
| M2 Boot chain | ☐ | | | |
| M3 Program → process | ☐ | | | |
| M4 Toolchain | ☐ | | | |
| M5 LFS to chroot | ☐ | | | |
| M6 Bootable system | ☐ | | | |
| M7 Break / fix | ☐ | | | |
| M8 Internals lab | ☐ | | | |
| M9 Rebuild | ☐ | | | |
| M10 Reflection | ☐ | | | |
