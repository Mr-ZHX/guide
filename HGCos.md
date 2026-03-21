# Operating System Course Labs — Experiment Overview

This document describes the **purpose and expected outcomes** of the experiments reflected in this repository (Linux 0.11–based kernel and user-mode work). It is written for readers who are **not** familiar with the assignments. It does **not** explain implementation steps, file edit lists, or grading rubrics.

---

## How many “labs” are in this project?

This repository’s `README.md` defines **eight separate lab tasks** (eight graded items in its task list). Counted by **official course experiment numbers** in that same document, they belong to **five chapters**: *Experiment 1* (two tasks), *Experiment 2* (one task), *Experiment 5* (two tasks), *Experiment 8* (two tasks), and *Experiment 11* (one task).

So: **8 tasks**, but **not** “Experiment 1 through Experiment 8” in the syllabus — the numbering **skips** 3, 4, 6, 7, 9, and 10 because those chapters are either not included in this repo’s checklist or are covered elsewhere in the full course materials. Below, each of the **eight tasks** is numbered **1–8** for quick reference, with the **official title** in parentheses.

---

## Context

The codebase is organized around **Linux 0.11** teaching kernels and small user programs. Several directories (`mission5817`, `mission5818`, …) hold **reference snapshots** aligned with individual lab milestones (e.g., one directory may correspond to “after Experiment 1 Task 1,” another to “after Experiment 8 Task 2”). The experiments collectively cover toolchain use, early boot behavior, process control primitives, paging-related introspection, and a minimal pseudo–file-system interface for process information.

---

## Experiment 1 of 8 — Using the experiment environment (syllabus: Lab 1, Task 1)

### First change inside the kernel

**What you do:** Modify the system so that, during normal startup, the kernel emits a simple greeting on the console.

**What you should understand:** How the teaching kernel is built and run, and where the first user-visible kernel initialization output is produced.

**Success looks like:** After boot, a short custom message appears in the kernel log / console output alongside (or instead of) the usual boot traces, confirming that your change is linked into the running image.

---

## Experiment 2 of 8 — Using the experiment environment (syllabus: Lab 1, Task 2)

### User programs, libraries, and a multi-file build

**What you do:** Work in **user space**: extend a sample application, add a small multi-source program (e.g., arithmetic in a separate module), and drive the build with **Make**.

**What you should understand:** How user programs link against the course libc stubs, how headers and `_syscall` macros are used, and how to structure a trivial project with separate compilation units.

**Success looks like:** The sample program prints your chosen text; the multi-file program reads input, calls your helper, and prints a correct result; `make` / `make clean` behave as expected.

---

## Experiment 3 of 8 — Operating system startup (syllabus: Lab 2)

**What you do:** Observe and adjust the **early boot path** (real mode / transition toward protected mode) and a small piece of **kernel driver output** related to storage setup.

**What you should understand:** The sequence from bootloader / setup code to the kernel proper; how BIOS-style text output differs from later kernel `printk` output; where partition / disk initialization messages originate.

**Success looks like:** A custom message appears during the setup phase (confirming your assembly hook runs), and hard-disk initialization messaging reflects the intended wording or formatting change.

---

## Experiment 4 of 8 — Process creation (syllabus: Lab 5, Task 1)

### `fork` and `wait`

**What you do:** Write a user program that creates a child with **`fork`**, uses **`wait`** so the parent synchronizes with the child’s exit, and prints **process identifiers** at well-defined points.

**What you should understand:** Parent vs. child after `fork`, ordering of execution without synchronization, and how `wait` reaps the child and affects control flow.

**Success looks like:** Console output shows distinct PIDs for parent and child, a clear “child exits → parent continues” ordering, and no runaway zombies under the simple scenario exercised by the lab.

---

## Experiment 5 of 8 — Process creation (syllabus: Lab 5, Task 2)

### `execve` and address space replacement

**What you do:** Provide two programs: one (“old”) that **`execve`s** into another (“new”), and the replacement that runs with a **new main image** but related identity semantics (PID behavior as defined by the course).

**What you should understand:** That `exec` replaces the program in the current process without necessarily creating a new PID; lines after a successful `exec` in the old program do not run.

**Success looks like:** Running the “old” program results in output from the “new” program; the “old” program’s post-`execve` path does not execute after a successful exec.

---

## Experiment 6 of 8 — Address mapping and memory sharing (syllabus: Lab 8, Task 1)

### Physical memory introspection from user space

**What you do:** Expose a **new system call** (or equivalent kernel entry) that reports **physical memory / page usage** statistics and exercises **allocate–free** of a physical page from the kernel’s perspective, callable from a tiny user test program.

**What you should understand:** The kernel’s page map (`mem_map`–style accounting), total pages vs. free pages, and the boundary between user mode and kernel mode for memory management information.

**Success looks like:** Invoking the test program prints counts (total pages, free vs. used) and shows the effect of allocating one page and then freeing it, as visible in the printed statistics.

---

## Experiment 7 of 8 — Address mapping and memory sharing (syllabus: Lab 8, Task 2)

### Virtual vs. physical address awareness

**What you do:** Run a user program that prints the **virtual (logical) address** of a stack or local variable and then **busy-waits** so the process remains runnable for inspection.

**What you should understand:** That user programs normally see **virtual addresses**; correlating them with physical frames requires paging structures and external tools (e.g., `/proc`-style views or debug monitors), not plain `%p` in user space alone.

**Success looks like:** The program prints a stable user-space address for the variable; while it spins, an observer (debugger, kernel monitor, or course-provided mechanism) can relate that logical address to paging / physical backing as required by the lab instructions.

---

## Experiment 8 of 8 — A minimal `proc`-style file system (syllabus: Lab 11)

**What you do:** Extend the kernel **virtual file system** so a special file type (e.g., under `/proc`) exposes a **tabular listing of processes** with fields such as PID, state, parent, scheduling counter, and start time. User programs read this node like a normal read-only file.

**What you should understand:** How character / block special files differ from regular files; how the kernel can synthesize file content from in-memory structures (`task_struct` table) instead of disk blocks.

**Success looks like:** After boot, the mount point and special node exist; reading the node returns a header line and one row per live task with the expected columns; repeated reads behave consistently with the course definition of offset / buffering.

---

## Summary Table

| # (this doc) | Syllabus label (`README`)     | Focus                         | Main artifact                          |
|--------------|-------------------------------|-------------------------------|----------------------------------------|
| 1            | 实验1 — 任务1                 | Environment & kernel edit     | Boot-time kernel message               |
| 2            | 实验1 — 任务2                 | User build system             | Sample app + multi-file `make` project |
| 3            | 实验2                         | Boot & early I/O / disk setup | Setup message + disk init output       |
| 4            | 实验5 — 任务1                 | `fork` / `wait`               | Parent–child PID trace program         |
| 5            | 实验5 — 任务2                 | `execve`                      | Old → new program transition           |
| 6            | 实验8 — 任务1                 | Paging accounting             | System call + physical memory stats    |
| 7            | 实验8 — 任务2                 | Virtual addresses             | Address print + spin for observation   |
| 8            | 实验11                        | Synthetic FS                  | `/proc`-like `psinfo` text node        |

---

*End of overview.*
