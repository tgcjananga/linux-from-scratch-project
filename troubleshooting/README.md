# troubleshooting/ — recovery runbook

Built in M7 from failures I **caused on purpose and fixed without reverting the snapshot**.
Reverting teaches nothing; recovering teaches the boot chain.

## Symptom index

Fill this in as each failure is done. This table is the actual deliverable — the individual
write-ups are its evidence.

| Symptom | Stage that failed | Likely cause | Recovery |
|---|---|---|---|
| `No bootable device` | firmware | | |
| GRUB `rescue>` prompt | bootloader | | |
| GRUB `error: file not found` | bootloader | | |
| `Kernel panic – not syncing: VFS: unable to mount root fs` | kernel | | |
| `Kernel panic – no init found` | kernel → PID 1 | | |
| Boots, then drops to emergency shell | init / fstab | | |
| Every command: `No such file or directory` (but the file exists) | dynamic loader | | |
| Services fail with unrelated errors | full filesystem | | |

## Per-failure write-up template

```markdown
# <what I broke>

**Break:** the exact command/edit that caused it.
**Symptom:** verbatim error text, and at which boot stage it appeared.
**Diagnosis:** what I checked, in order — including the dead ends.
**Recovery:** the exact steps, from a rescue shell / live ISO / GRUB command line.
**Prevention:** what I'd do differently, and whether a script in `scripts/` can catch it.
**Time to fix:** (worth tracking — it drops sharply as the boot chain becomes familiar)
```

Record the **dead ends**. A runbook that only shows the correct path is useless at 1 AM;
knowing what *doesn't* narrow the problem is half the value.
