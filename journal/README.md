# journal/ — build log

Chronological record of the build. Chapter-per-file, and a per-package entry inside each.

Naming: `00-environment.md`, `ch04-preparing.md`, `ch05-temp-toolchain.md`, … ,
`ch11-bootloader.md`, `rebuild.md`.

## Per-package entry template

```markdown
### <package> <version>

1. **What is it?**
2. **Why here, in this order?**   (what would fail if it came later?)
3. **What did it install?**       (from `scripts/file-manifest.sh` — measured, not guessed)
4. **How do I verify it?**        (command + expected output)

**Build time:** Xm Ys      **Log:** `logs/<package>.log`
**Problems:** (anything that failed, and what fixed it)
```

Question 2 is the one that teaches the most — the LFS package order is not arbitrary, and
reconstructing the reason is most of the understanding.

## Rules

- Write the entry **while the package builds**, not afterwards from memory
- Record failures verbatim, including the ones caused by my own mistakes — M7 and M9 are
  built out of these
- Note the snapshot name taken before each chapter
