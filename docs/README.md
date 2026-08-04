# docs/ — theory track

Concepts written in **my own words**. Never a copy of the LFS book; if I can't restate it,
I haven't learned it yet.

Rule: every factual claim here is backed by a command, and that command appears in the doc
alongside its output.

| File | Milestone | Status |
|---|---|---|
| `linux-architecture.md` | M1 | ☐ |
| `filesystem-hierarchy.md` | M1 | ☐ |
| `linux-distributions.md` | M1 | ☐ |
| `storage.md` | M1 | ☐ |
| `filesystems.md` | M1 | ☐ |
| `linux-boot-process.md` | M2 | ☐ |
| `elf-format.md` | M3 | ☐ |
| `process-management.md` | M3 | ☐ |
| `memory-management.md` | M3 | ☐ |
| `toolchain.md` | M4 | ☐ |
| `chroot.md` | M5 | ☐ |
| `kernel.md` | M6 | ☐ |
| `grub.md` | M6 | ☐ |
| `syscall-boundary.md` | M8 | ☐ |
| `reflection.md` | M10 | ☐ |
| `next-gentoo.md` | M10 | ☐ |

## Suggested shape for each doc

```markdown
# <Topic>

## The question this answers

## Explanation (my own words)

## Evidence
$ command
<actual output from my machine>

## What breaks if this is wrong

## Open questions
```

The **"What breaks if this is wrong"** section is the most valuable one — it is what turns
into `troubleshooting/` later.
