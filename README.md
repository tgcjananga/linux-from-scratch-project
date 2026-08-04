# linux-from-scratch-project

A hands-on journey through Linux From Scratch to understand Linux internals, operating
system concepts, toolchains, the boot process, the kernel, and system architecture by
building a Linux system completely from source.

The end goal isn't "LFS booted once" — it's a **rebuild I can do from my own notes**
(milestone M9), which is what makes the follow-on Gentoo project a set of deliberate
choices rather than a black box.

## Start here

**[ROADMAP.md](ROADMAP.md)** — 11 milestones, each with testable exit criteria. Progress
table lives at the bottom of that file.

Work runs in two coupled tracks: a **theory track** (`docs/`, `diagrams/`) and an
**implementation track** (`experiments/`, `journal/`, `scripts/`, `configs/`). Theory leads
implementation by one milestone — read about the boot chain *before* having to debug it.

## Layout

| Path | What's in it |
|---|---|
| [ROADMAP.md](ROADMAP.md) | Milestones, exit criteria, standing rules |
| [docs/](docs/) | Concepts in my own words, each claim backed by a command |
| [diagrams/](diagrams/) | Boot chain, address space, toolchain flow |
| [experiments/](experiments/) | Small programs, one question each, with observed output |
| [journal/](journal/) | Build log per chapter and package; the rebuild diary |
| [configs/](configs/) | `kernel.config`, `grub.cfg`, `fstab` — the files that define the system |
| [scripts/](scripts/) | Verification + build automation (the package manager LFS lacks) |
| [troubleshooting/](troubleshooting/) | Symptom-indexed recovery runbook from real failures |
| `logs/` | Raw per-package build output (gitignored; evidence for later debugging) |

Each directory has its own `README.md` with the conventions and templates for what goes in it.

## Status

Pre-M0. See the progress table in [ROADMAP.md](ROADMAP.md#progress).
