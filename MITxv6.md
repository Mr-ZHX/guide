# xv6-riscv — Lab / Coursework Framework Overview (English)

This document describes **what is actually in this `xv6-riscv/` repository** for someone who has never used xv6. It focuses on **repository structure** and **what subsystems exist**, not on solution strategies.

**Critical fact about this tree:** There are **no separate `lab1/`, `lab2/`, … folders** and **no assignment PDFs** checked in. The codebase is a **single, complete teaching OS** (kernel + userland + build system). University courses (commonly MIT **6.1810** / older **6.828**, or similar) typically assign **weekly labs** that ask you to **edit this same tree**; the lab boundaries live in **external handouts**, not in directory names here.

The in-tree `README` points to MIT’s **6.1810** resources as context for xv6.

---

## What you “have” in one snapshot

| Area | Role |
|------|------|
| **`kernel/`** | The xv6 **kernel**: boot, traps, RISC-V glue, physical memory, virtual memory, processes, scheduler, system calls, file system, log, pipes, `exec`, devices (UART, virtio block, PLIC interrupt controller), locking, console. |
| **`user/`** | **User programs** and the **user C library** (`ulib`, `printf`, `umalloc`, `user.h`, `usys.pl` stubs). Includes the shell (`sh.c`), utilities (`cat`, `ls`, …), stress tests (`usertests.c`, `grind.c`, …). |
| **`mkfs/`** | Host-side tool to build the **file-system image** embedded in the kernel boot. |
| **`Makefile`** | Builds the kernel, user binaries, filesystem image, and **`make qemu`** to run under QEMU. |
| **`test-xv6.py`** | Python helper related to automated testing (exact use depends on your course scripts). |

So: the “framework” is **the whole OS**, not a chopped-up per-lab SDK.

---

## Kernel subsystems (where coursework changes usually land)

These are **modules**, not official lab numbers:

- **Boot & low-level RISC-V:** `entry.S`, `start.c`, `kernelvec.S`, `trampoline.S`, `riscv.h`.
- **Memory:** `kalloc.c` (physical pages), `vm.c` / `vm.h` (page tables, `copyin`/`copyout`), `memlayout.h`.
- **Concurrency:** `spinlock.*`, `proc.c` / `proc.h`, `swtch.S`, `main.c`.
- **Traps & syscalls:** `trap.c`, `syscall.c`, `sysproc.c`, `sysfile.c`, `syscall.h`.
- **I/O & storage:** `bio.c`, `fs.c`, `log.c`, `file.c`, `pipe.c`, `exec.c`, `virtio_disk.c`, `console.c`, `uart.c`, `plic.c`.
- **User/kernel ABI:** `user/usys.pl` (generates syscall stubs), `kernel/syscall.c` dispatch table.

**`user/usertests.c`** is a large **integration test suite** for syscalls, VM safety (`copyin`/`copyout` tests), file system, pipes, and more—courses often require it to pass after certain labs.

---

## Userland “assignments” implied by programs (not separate labs)

The `user/` directory is a **collection of small programs** used for demos and grading:

- Everyday tools: `cat`, `echo`, `grep`, `ls`, `mkdir`, `rm`, `wc`, etc.
- **Shell** (`sh.c`) for interactive use.
- **Stress / correctness:** `usertests`, `grind`, `forktest`, `stressfs`, `logstress`, orphan tests, etc.

Adding or changing a utility is a **typical early project** in xv6-based courses, but this repository does not label which program belongs to which week.

---

## Relationship to “labs” in a real course

Because **this clone does not name labs**, the following is **not** extracted from your files:

- Official **lab titles**, **due dates**, or **grading scripts** (`grade-lab-*`, `gradelib.py`, etc. are **not** present here).
- A **unique** mapping from “Lab *n*” → files (that mapping is on the course website each term).

**What you should do as a student:** Use your instructor’s **lab PDF / autograder** for that term. This tree is the **shared baseline** you modify.

---

## Typical *themes* in MIT-style xv6 courses (reference only)

> **Disclaimer:** Topics and order **change by year and school**. Treat this as a **rough** map of what xv6 labs *often* cover—not a claim about *your* repository or syllabus.

Common themes include:

1. **User-level tools** — implement or extend small programs in `user/` using the existing syscall surface.
2. **System calls** — add kernel support + user stubs for new syscalls.
3. **Virtual memory / page tables** — inspect or change `vm.c`, page-table layouts, or lazy behaviors.
4. **Traps & timer / alarm-style features** — work in `trap.c` and related code.
5. **Copy-on-write fork** — change `fork`, `vm`, page faults, or reference counting.
6. **File system features** — e.g. larger files, `mmap`, or consistency tweaks (`fs.c`, `log.c`, etc.).
7. **Concurrency / locking** — reduce races or add synchronization in kernel or user tests.
8. **Devices / I/O** — sometimes network or driver labs touch virtio or other device code.

Your term may use a **subset** or **different** sequence; rely on the **official handout**, not this list.

---

## Build & run (what the tree expects)

From `README`: a **RISC-V GCC toolchain** and **QEMU** for `riscv64` (`make qemu`). The `Makefile` enforces a **minimum QEMU version** (see `MIN_QEMU_VERSION`). No implementation steps are given here.

---

## What this document omits

- **Algorithms** for VM, FS, or scheduling.
- **Any student lab report** or score claims.
- **Line-by-line** behavior of every test in `usertests.c`.

If your course ships an **extra** directory (e.g. `gradelib/`, `conf/lab.mk`) in another branch, merge that material into your tree and extend this overview accordingly.
