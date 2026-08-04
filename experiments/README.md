# experiments/ — implementation track

Each directory answers **one question** with code I can run. Not a tutorial collection — a
lab notebook.

## Convention

Every experiment directory contains:

```
README.md     the question, the method, the OBSERVED output, and the conclusion
Makefile      `make` builds, `make run` runs, `make clean` cleans
*.c           the code
```

`README.md` template:

```markdown
# <experiment name>

**Question:** what am I actually trying to find out?

**Method:** what I built and how I observed it.

**Observed:**
$ make run
<real pasted output>

**Conclusion:** the one sentence I couldn't have written before running this.

**Surprise:** anything that contradicted what I expected. (Often the real finding.)
```

The **Surprise** field matters most. If an experiment produced no surprise, either it was
too easy or I wasn't paying attention.

## Directories

| Dir | Milestone | Question |
|---|---|---|
| `elf/` | M3 | How does a C file become a loadable binary, and what's inside it? |
| `processes/` | M3 | How do processes get created, related, and reaped? |
| `memory/` | M3 | What does a process's address space actually contain? |
| `toolchain/` | M4 | Can I prove a cross-compiler targets something other than this host? |
| `process/` | M8 | How do processes talk to each other? (pipes, IPC, shared memory) |
| `network/` | M8 | What does a TCP connection look like from both sides of the syscall? |
| `kernel/` | M8 | Can I run my own code in kernel space, on a kernel I compiled? |
