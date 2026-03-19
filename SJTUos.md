# IPADS OS Course Lab Manual (English)

This document is an English overview of the Operating System course lab series designed by the Institute of Parallel and Distributed Systems (IPADS), Shanghai Jiao Tong University. The labs use the **ChCore** microkernel and target **Raspberry Pi 3B+** (QEMU emulation or real hardware). After completing the series, you can run applications such as GBA games, DeepSeek, or Qwen-1.5b on your DIY ChCore kernel.

---

## Table of Contents

1. [Overview](#overview)
2. [Environment and Getting Started](#environment-and-getting-started)
3. [Lab 0: Defuse the Bomb (ARM Assembly)](#lab-0-defuse-the-bomb-arm-assembly)
4. [Lab 1: Kernel Booting](#lab-1-kernel-booting)
5. [Lab 2: Memory Management](#lab-2-memory-management)
6. [Lab 3: Process and Thread](#lab-3-process-and-thread)
7. [Lab 4: Multicore Scheduling and IPC](#lab-4-multicore-scheduling-and-ipc)
8. [Lab 5: Virtual File System](#lab-5-virtual-file-system)
9. [Lab 6: GUI (Optional)](#lab-6-gui-optional)
10. [Grading and Submission](#grading-and-submission)

---

## Overview

- **Platform**: ARM64 (aarch64), Raspberry Pi 3B+
- **Kernel**: ChCore microkernel (see [ChCore at ATC’20](https://www.usenix.org/conference/atc20/presentation/gu))
- **Build / run**: Docker-based; Linux host recommended. Dev-Container is supported for a consistent environment.
- **Documentation**: Detailed Chinese lab pages and source-code appendices are in the `Pages/` directory; this document summarizes all labs in English.

---

## Environment and Getting Started

- **Docker** is required; labs are intended to run on **Linux**. For Windows/macOS, use the provided **Dev-Container** or a **VMware** VM image.
- **Line endings**: Use **LF** (not CRLF) for scripts and Makefiles.
- **Do not run builds with `sudo`**; timestamp files may break subsequent builds.
- **Exercise types** in lab docs:
  - **Thought questions**: Answer in your report (text or figures).
  - **Coding exercises**: Fill in code in ChCore and describe your implementation in the report; these carry most of the grade.
  - **Challenge exercises**: Harder optional tasks for deeper understanding.

CI (GitHub Actions / GitLab CI) can grade your work; required files are determined by `filelist.mk` in each lab.

---

## Lab 0: Defuse the Bomb (ARM Assembly)

**Goal**: Learn ARM assembly and basic debugging by “defusing” a binary bomb. Inspired by the CSAPP bomb lab, but for **ARM64** to prepare for later ChCore/Raspberry Pi work.

### Contents

- **Part 1**: Basics — ARM assembly, QEMU (user-mode), and GDB.
- **Part 2**: Analyze the bomb binary and provide correct inputs so the program exits normally (all phases defused).

### Key files

- `Lab0/bomb.c`: Reference source; the actual bomb is generated from `student-number.txt`.
- `Lab0/scripts/generate_bomb.sh`: Generates the bomb binary using the student number.
- **Submit**: `ans.txt` (one line per phase) and `student-number.txt`.

### Makefile targets

| Target        | Description                                              |
|---------------|----------------------------------------------------------|
| `make bomb`   | Generate bomb using `student-number.txt`                 |
| `make qemu`   | Run bomb under QEMU aarch64                             |
| `make qemu-gdb` | Run QEMU with GDB server (port 1234)                  |
| `make gdb`    | Connect GDB to QEMU for debugging                       |
| `make grade`  | Run grader (Docker)                                     |
| `make submit` | Package `ans.txt` and `student-number.txt` for submit   |

### Grading (scores.json)

- Phase 1–4: 15 points each.
- Phase 5: 20 points.
- All phases defused message: 20 points.  
**Total**: 100 points.

> **Warning**: Fill in `student-number.txt` before submission; otherwise the lab may be scored zero.

---

## Lab 1: Kernel Booting

**Goal**: Understand and implement early kernel boot: exception level setup, kernel page tables, and enabling the MMU. This is the first ChCore kernel lab.

### Structure

1. **RTFSC (1)**: Code walkthrough — ChCore build system; no exercises.
2. **Machine boot**: aarch64 boot registers and main boot functions.
3. **Page table setup**: aarch64 page table format and Raspberry Pi 3 memory layout; implement kernel page table configuration.

### Student tasks (code)

- **tools.S**: TODO 1 — read current exception level; TODO 2 — set return address and exception level for EL3→EL2; TODO 4 — another boot path.
- **uart.c**: TODO 3 — UART output (e.g., for early debug).
- **mmu.c**: TODO 5 — set L0/L1 entries and map physical memory regions (e.g., `PHYSMEM_START`–`PERIPHERAL_BASE` and `PERIPHERAL_BASE`–`PHYSMEM_END`) with 2 MB granularity.

### Running

- Use QEMU or a real Raspberry Pi 3B+. After solving the exercises, the kernel enters the ChCore shell (e.g., run `hello_world.bin`, `ls`).

### Grading

- One main criterion: kernel reaches main and prints “Welcome to ChCore shell!” (100 points, userland test).

> **Important**: Read the kernel debugging appendix before starting.

---

## Lab 2: Memory Management

**Goal**: Implement physical memory management (buddy + slab), kernel page table allocation, and page fault handling (on-demand allocation and copy-on-write).

### Structure

1. **Physical memory management**: Implement **buddy system** and **slab allocator**.
2. **Virtual page table management**: Page table allocation and page table entry permissions; implement page table allocation functions.
3. **Page fault handling**: aarch64 exception handling; implement demand paging and copy-on-write according to page table configuration.

### Student tasks (code)

- **buddy.c**: TODO 1 — `split_chunk`, `merge_chunk`, `buddy_get_pages`, `buddy_free_pages`.
- **slab.c**: TODO 2 — slab allocation/free logic.
- **kmalloc.c**: TODO 3 — use buddy/slab for `kmalloc`/kfree.
- **page_table.c** (aarch64): TODO 4 — map/unmap and page table helpers.
- **pgfault.c** (aarch64): TODO 5 — route page fault to handler.
- **vmspace.c**: TODO 6 — virtual space/page table setup.
- **pgfault_handler.c**: TODO 7 — handle page faults (e.g., demand paging, copy-on-write).

### Grading (scores.json)

- Buddy: allocate/free order 0; each order; all orders; all memory (5 pts each).
- kmalloc: 10 pts.
- Map/unmap: one page, multiple pages (10 pts each); huge range (20 pts).
- Physical memory computation: 1 + 1 + 3 pts.
- Page fault (userland “Welcome to ChCore shell!”): 30 pts.

---

## Lab 3: Process and Thread

**Goal**: Create the first user-mode process and thread, complete exception handling and system calls, and run a Hello-World program on ChCore.

### Structure

1. **RTFSC (2)**: ChCore microkernel mechanisms and user/kernel interaction.
2. **Thread management**: Create first user process/thread; understand kernel-to-user transition.
3. **Exception handling**: Implement exception handling and necessary support.
4. **System calls**: Implement the required system calls so user programs can run and produce output.
5. **User program**: Write a simple user program with ChCore libc, compile it, and add it to the kernel image.

### Preparation

From Lab 3 onward, userland code is used. Run:

```bash
git submodule update --init --recursive
```

to fetch libc and other submodules.

### Student tasks (code)

- **context.c** (aarch64): LAB 3 TODO — set up user context (e.g., PC, SP).
- **irq_entry.S**: LAB 3 TODO — save/restore state and dispatch exceptions/syscalls.
- **thread.c**: LAB 3 TODO — thread creation and user context setup.
- **cap_group.c**: LAB 3 TODO — capability group creation and setup for the first process.
- **stdio.c** (chcore-port): LAB 3 TODO — port printf or similar for userland.

### Grading

- Cap create pretest: 20 pts.
- Thread create (root thread) pretest: 20 pts.
- “Hello Userland!”: 20 pts.
- “Welcome to ChCore shell!” (printf): 20 pts.
- “Hello ChCore!” (userland app): 20 pts.

> **Tip**: If tests fail due to timeouts, adjust the `TIMEOUT` variable in the lab’s Makefile.

---

## Lab 4: Multicore Scheduling and IPC

**Goal**: Enable multi-core boot, implement round-robin scheduling on multiple cores, and implement ChCore’s **inter-process communication (IPC)** based on capabilities. Optionally optimize IPC performance.

### Structure

1. **Multicore boot**: Use Raspberry Pi firmware to wake secondary cores.
2. **Multicore scheduling**: Round-robin scheduling across cores.
3. **IPC**: Implement capability-based IPC.
4. **IPC tuning**: Optimize IPC for the given benchmarks.

### Student tasks (code)

- **policy_rr.c**: Exercise 1 — enqueue; 2–4 — scheduler logic (pick thread, tick, etc.).
- **sched.c**: Exercise 3 — enqueue/placement; 4 — scheduling decision.
- **timer.c** (arch and generic): Exercise 5–6 — timer and preemption.
- **connection.c**: Exercise 7 — IPC connection setup, send/receive, and capability handling.

### Grading

- Scheduler init: 10 pts.
- Scheduler enqueue: 10 pts.
- Scheduler dequeue: 10 pts.
- Cooperative scheduling test: 20 pts.
- Preemptive scheduling test: 20 pts.
- IPC test: 30 pts.

Code to fill in is between `/* LAB 4 TODO BEGIN (exercise #) */` and `/* LAB 4 TODO END (exercise #) */`.

---

## Lab 5: Virtual File System

**Goal**: Implement the VFS abstraction so that different file systems (e.g., ext4, tmpfs, FAT32) can be used under a unified API. ChCore uses **FSM** (file system manager) and **FS_Base** to integrate file systems; you implement this layer and a page-fault optimization (Boweraccess).

### Structure

1. **POSIX adaptation**: How ChCore provides a POSIX-like file API.
2. **FSM**: Page cache, mount point management, path resolution — implement the FSM forwarding layer.
3. **FS_Base**: Common library that wraps standard file operations for file system processes; implement this layer.
4. **Boweraccess**: Optimize file system page-fault handling using prefetch (FS_Page_Fault).

**Note**: Lab 5 involves **only userland code** under the `user/` directory; no kernel code to write.

### Grading

- Exercises are in FSM, FS_Base, and Boweraccess; see the lab’s `scores.json` (if present) and the online lab pages for point breakdown.

Fill-in code is between `/* LAB 5 TODO BEGIN (exercise #) */` and `/* LAB 5 TODO END (exercise #) */`.

---

## Lab 6: GUI (Optional)

**Goal**: Understand the Wayland-based GUI on ChCore (protocol, compositor) and write a small GUI application using the ChCore GUI framework.

This lab is optional and is not summarized in detail here; refer to the lab documentation and slides for Wayland and the compositor.

---

## Grading and Submission

- **Lab 0**: Submit `ans.txt` and `student-number.txt` (e.g., via `make submit`). Grading: `make grade` (Docker).
- **Labs 1–5**: Implement all TODOs marked in the code; run the provided tests. CI can merge your submitted files with the mainline repo (using `filelist.mk`) and run the same grader.
- **Report**: For each lab, answer thought questions and describe your coding solutions as required by the lab instructions.

---

## Quick Reference

| Lab | Theme                    | Main deliverables                                      |
|-----|--------------------------|--------------------------------------------------------|
| 0   | ARM assembly & bomb      | `ans.txt`, `student-number.txt`                       |
| 1   | Kernel boot & MMU        | Boot path, page table init (tools.S, mmu.c, uart.c)   |
| 2   | Memory management        | Buddy, slab, page table, page fault handler           |
| 3   | Process & thread         | Context, exception/syscall, first user process        |
| 4   | Multicore & IPC          | Multi-core boot, RR scheduler, IPC                    |
| 5   | VFS                      | FSM, FS_Base, Boweraccess (userland only)             |
| 6   | GUI (optional)           | Wayland-based GUI app                                 |

---

*This manual is derived from the IPADS OS Course Lab (Chinese) and summarizes structure, tasks, and grading for Labs 0–6. For detailed steps, tutorials, and source-code analysis, see the [official lab pages](https://sjtu-ipads.github.io/OS-Course-Lab/) and the `Pages/` directory in this repository.*
