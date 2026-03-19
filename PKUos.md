# XV6-Labs-2023 — Util Lab (English Lab Manual)

**Course**: Peking University (PKU) Operating Systems (Fall 2023) — XV6 Lab  
**Repository**: `xv6-labs-2023`  
**Current configuration**: `conf/lab.mk` sets `LAB=util`

This document is an English lab manual for the **util** portion of this XV6-Labs repository. It focuses on the user-space utilities and the required kernel support, based on what is present in this directory:

- Grading script: `grade-lab-util`
- User programs: `user/sleep.c`, `user/pingpong.c`, `user/primes.c`, `user/find.c`, `user/xargs.c`
- Kernel syscalls: `kernel/sysproc.c` (`sys_sleep`, `sys_uptime`) and syscall dispatch table in `kernel/syscall.c`

---

## Table of Contents

1. [Lab Overview](#lab-overview)
2. [Objectives](#objectives)
3. [Relevant Code in This Repository](#relevant-code-in-this-repository)
4. [User Programs and Expected Behaviors](#user-programs-and-expected-behaviors)
5. [Kernel Syscalls for Util](#kernel-syscalls-for-util)
6. [Build and Run](#build-and-run)
7. [Grading and Test Cases](#grading-and-test-cases)
8. [Notes on `time` Test](#notes-on-time-test)
9. [Submission Checklist (Typical)](#submission-checklist-typical)

---

## Lab Overview

The **util** lab extends XV6 with a set of user-level programs and required system calls/utilities so that basic user space functionality works correctly under QEMU. The utilities are then validated by the provided grading script.

In this repository, the current working tree is aligned with `LAB=util`, and the provided grade script is `./grade-lab-util`.

---

## Objectives

After completing the util lab, the following should work:

- `sleep` works correctly both with and without arguments, and actually uses the `sys_sleep` syscall for `sleep <n>`.
- `pingpong` prints the expected “received ping/pong” sequence using pipes and fork.
- `primes` generates primes using a classic pipeline of fork+pipe processes.
- `find` recursively searches files/directories starting from a given directory and prints matching paths.
- `xargs` reads arguments from stdin and repeatedly executes a command with the collected tokens.
- the grader’s `time` check succeeds (via the harness environment).

---

## Relevant Code in This Repository

### Grader

- `grade-lab-util`  
  Defines tests for:
  - `sleep` (no args / returns / makes syscall)
  - `pingpong`
  - `primes`
  - `find` (current directory / recursive)
  - `xargs`
  - `time` (host-side check)

### Kernel

- `kernel/sysproc.c`
  - `sys_sleep()`
  - `sys_uptime()`
- `kernel/syscall.c`
  - system call dispatch table includes:
    - `[SYS_sleep] sys_sleep`
    - `[SYS_uptime] sys_uptime`
- `kernel/syscall.h`
  - `SYS_sleep  = 13`
  - `SYS_uptime = 14`

### User programs

- `user/sleep.c`
- `user/pingpong.c`
- `user/primes.c`
- `user/find.c`
- `user/xargs.c`
- `user/usys.pl`
  - generates syscall stubs (includes `sleep` and `uptime`)

---

## User Programs and Expected Behaviors

### `sleep`

- File: `user/sleep.c`
- Behavior (as implemented):
  - If the program is run without exactly two arguments (e.g., `sleep`), it prints usage and exits without crashing.
  - If the argument exists, it calls the syscall wrapper `sleep(atoi(argv[1]))`.

Why this matters:

- The grader checks:
  1. `sleep` without arguments does not fail to execute.
  2. `sleep <n>; echo OK` continues and prints `OK`.
  3. `sleep <n>` actually reaches the kernel syscall by using `stop_breakpoint('sys_sleep')`.

### `pingpong`

- File: `user/pingpong.c`
- Behavior:
  - Creates two pipes.
  - Forks:
    - Child reads from one pipe and prints `"<pid>: received ping"` then writes to the other pipe.
    - Parent writes and reads `pong` similarly.

Grader expectations:

- Output must match:
  - `^\d+: received ping$`
  - `^\d+: received pong$`
  - and then `^OK$`.

### `primes`

- File: `user/primes.c`
- Behavior:
  - Builds a prime-filter pipeline using recursion (`proceed`).
  - Each stage:
    - reads the first integer from its input pipe as the prime,
    - prints it as `prime <p>`,
    - forks a child to continue the pipeline with the next pipe,
    - forwards numbers not divisible by `<p>`.

Grader expectations:

- Must print prime numbers:
  - `prime 2`, `prime 3`, `prime 5`, ... up to `prime 31`
  - then execute `echo OK`.

### `find`

- File: `user/find.c`
- Behavior:
  - Usage: `find [dirname] [pattern]`
  - Recursively traverses directories starting at `[dirname]`.
  - If an entry is a file/device and its short name matches `[pattern]`, it prints the full path.
  - If an entry is a directory, it recurses into it.

Grader expectations:

- `find . <needle>` must print:
  - exactly all matching paths in the current directory.
- `find . <needle>` must print:
  - all matching paths across multiple subdirectories.

### `xargs`

- File: `user/xargs.c`
- Behavior:
  - Reads stdin character-by-character.
  - Splits tokens by spaces and executes the given command at each newline.
  - For `xargs grep hello`, it executes `grep hello <token1> <token2> ...` for each produced line.

Grader expectations:

- The test runs `sh < xargstest.sh` and asserts that `hello` appears **exactly 3 times** in the output.

---

## Kernel Syscalls for Util

### `sys_sleep`

- File: `kernel/sysproc.c`
- Key behavior:
  - `argint(0, &n)` reads the first argument.
  - Negative `n` is clamped to 0.
  - Uses the `ticks` counter:
    - acquires `tickslock`
    - waits until `ticks - ticks0 >= n` while sleeping on `ticks`
    - returns `-1` if the process is killed during sleep

### `sys_uptime`

- File: `kernel/sysproc.c`
- Key behavior:
  - Returns the current `ticks` value to user space.

### Syscall dispatch

- File: `kernel/syscall.c`
- `sys_sleep` and `sys_uptime` are registered in the `syscalls[]` table via `SYS_sleep` and `SYS_uptime`.

### User syscall stubs

- File: `user/usys.pl`
- Stub generation includes:
  - `entry("sleep");`
  - `entry("uptime");`

---

## Build and Run

### Prerequisites (Toolchain and QEMU)

To build and run XV6 under QEMU you typically need:

- A RISC-V `newlib`/ABI toolchain (installed on the host and available in your `PATH`, e.g. `riscv64-unknown-elf-gcc` / `riscv64-unknown-elf-objdump`).
- QEMU compiled for RISC-V: `qemu-system-riscv64` supporting `riscv64-softmmu`.

The repository README describes the general requirement: install a RISC-V toolchain from the official riscv-gnu-toolchain repository, and build/install QEMU that supports running `make qemu`.

### Build the XV6 image

Typical flow (the Makefile already supports this):

1. Configure toolchain and QEMU as required by XV6.
2. Build for the current lab:

```bash
make
```

### Run in QEMU (base usage)

The xv6 README indicates you can run:

```bash
make qemu
```

### Run grading

To test the current util solution (aligned with `LAB=util`):

```bash
make grade
```

The Makefile cleans and then runs:

```bash
./grade-lab-util
```

---

## Grading and Test Cases

The grader defines the following tests (with points):

- `sleep, no arguments` — 5 pts
- `sleep, returns` — 5 pts
- `sleep, makes syscall` — 10 pts (uses a breakpoint at `sys_sleep`)
- `pingpong` — 20 pts
- `primes` — 20 pts
- `find, in current directory` — 10 pts
- `find, recursive` — 10 pts
- `xargs` — 19 pts
- `time` — 1 pt

The runner first builds (`make()` in `gradelib.py`), resets the filesystem image, then runs each test via QEMU and pattern matches output.

---

## Notes on `time` Test

The `time` test does not rely on a `user/time.c` program in this repository. Instead, `gradelib.py` reads a host-side file:

- `time.txt`

It expects `time.txt` to contain a single integer (`^\d+$`), interpreted as “number of hours spent on the lab”.

This file is usually created by the course/harness environment outside the guest OS execution. For local grading reproduction, ensure `time.txt` exists and is valid.

---

## Submission Checklist (Typical)

Since the exact submission rules depend on your course setup, use this checklist as a typical guide:

- Ensure `make grade` passes for util.
- No debug/temporary files should break the build.
- Your changes should include all required user programs and syscall support for the util tests:
  - `sleep` + `sys_sleep`
  - pipes/fork/exec as already required by these utilities
  - `find` directory traversal correctness
  - `xargs` parsing and exec loop correctness

---

End of document.

