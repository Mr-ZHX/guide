# TAOS — Coursework / Framework Overview (English)

This document is for someone who has **never** seen the project: what the **repository contains**, what the **system is trying to be**, and which **functional areas** the code is organized into. It is derived **only** from files present under `TAOS/` (mainly `README.md`, `Makefile`, and `kernel/src/**`).

**Important:** This tree **does not** ship separate weekly handouts or folders named like `lab1`, `lab2`, etc. There is **one** integrated kernel codebase. The sections below therefore describe **major subsystems** (how the framework is partitioned in source) rather than officially numbered labs. External planning (e.g. GitHub Projects / wiki) is referenced in `README.md` but is **not** part of this clone.

---

## What TAOS is (from `README.md`)

**TAOS** (*Totally Awesome Operating System*) is presented as a **multicore / distributed** x86-oriented OS developed in the context of **UT Austin CS 378: Multicore Operating Systems**. High-level goals stated in the README include:

- Processes that can **spawn processes on remote machines** also running TAOS.
- **Hiding distribution** from users: automatic handling of **distributed memory**, **distributed filesystem** behavior, and **routing IPC** to remote hosts when appropriate.

So the “experiment” in the large is: **build and extend a research-style OS** with local and distributed semantics, not a sequence of small isolated lab zips.

---

## Repository layout (what exists here)

| Path | Role |
|------|------|
| `kernel/` | The **no_std** Rust kernel (main artifact). Build/run/test via Cargo from this directory. |
| `kernel/src/lib.rs` | Declares top-level modules: `constants`, `devices`, `events`, `filesys`, `init`, `interrupts`, `ipc`, `logging`, `memory`, `net`, `processes`, `syscalls`. |
| `resources/executables/` | Small **user programs** (e.g. C sources) used with the filesystem image workflow. |
| `setup_sysroot.sh` | Host setup helper (sysroot). |
| `Makefile` | Wraps **build**, **run** (GUI / terminal / gdb modes), **test**, **fmt**, **clippy**, and **blank_drive** (EXT2 image preparation). |

**External documentation** (not in-repo): README links to a **GitHub wiki** and **GitHub Projects** for deeper specs and timelines.

---

## Tooling and verification (what you “do” with the framework)

From `README.md` and `Makefile`:

- **Dependencies:** QEMU (README specifies **9.2**), and **Limage** from a linked Git repo (not necessarily the crates.io version).
- **Run:** `make run` (GUI), `make run-term` (serial/console-oriented).
- **Host tests:** `make test` → `cargo test` inside `kernel/`.
- **Quality:** `make check` (clippy with warnings denied + `rustfmt` check), `make fmt`.
- **Disk image:** `make blank_drive` creates/formats a storage image and can embed `../resources` for EXT2-backed experiments.

No step-by-step implementation guide is included in this repository.

---

## Functional areas (how the kernel framework is decomposed)

These are **code modules**, treated here as the natural “units of work” a course or team might parallelize. Names match `kernel/src/`.

### `init/`

Boot-time bring-up and orchestration of subsystems (entry into the rest of the kernel).

### `interrupts/`

Low-level x86 interrupt plumbing: e.g. **GDT**, **IDT**, **I/O APIC** / **x2APIC** integration—hardware-facing control for delivering interrupts to the kernel.

### `memory/`

Physical/virtual memory infrastructure: frame allocation strategies (**boot**, **bitmap**, **buddy**), **paging**, **TLB** concerns, **page faults**, kernel **heap**, and higher-level **MM** abstractions. Unit tests exist under several of these files.

### `processes/`

**Process** representation, **CPU register** images, **program loading**, and **test harness binaries** (assembler snippets under `processes/test_binaries/`). Integration-style kernel tests schedule processes and await completion (see `processes/mod.rs` and `syscalls/mod.rs`).

### `syscalls/`

User/kernel boundary: **syscall dispatch** plus topic-specific handlers such as **`fork`**, **memory mapping (`memorymap`)**, and **sockets**. Kernel tests exercise behaviors like exit codes, print-then-exit, fork/wait, and fork with **copy-on-write** expectations (as documented in test comments).

### `events/`

Kernel **scheduling** and **async** execution: task contexts, cooperative points, futures-based waiting on devices/processes, sleep, synchronization primitives, and an **event runner** driving work.

### `ipc/`

Inter-process and distributed-oriented **IPC** scaffolding: channels, message types, serialization, FD tables, namespaces, mount manager hooks, single-producer/single-consumer structures, etc.—aligned with the README’s theme of **routing** communication in a potentially distributed setup.

### `filesys/` (EXT2)

An **ext2** implementation path: block I/O, caching, inodes, directory tree, allocator, IDE-backed storage, and filesystem-level tests.

### `net/`

Network stack integration using **smoltcp**, DHCP, sockets, and coupling to a **USB Ethernet (ECM)** device path under `devices/xhci/`.

### `devices/`

Hardware support breadth: **PCI**, **MMIO**, **serial**, **PS/2** keyboard/mouse, **framebuffer / text rendering**, **xHCI** (USB, including ECM), **SD card**, **HDA audio** (DMA, buffers, wav parsing, etc.).

### `logging/`

Kernel logging support used across subsystems.

### `constants/`

Shared numeric/syscall/device constants consumed by other modules.

---

## User-space samples (`resources/`)

The tree includes tiny **C** programs (e.g. `hello.c`, `ret.c`) intended to live on the filesystem image built via the Makefile workflow. They support end-to-end testing of **loading executables** and basic **process behavior**, complementing the assembler tests embedded in the kernel tree.

---

## What this document omits

- **Implementation recipes** (algorithms, data structures, locking protocols).
- **Wiki / Projects** content (not vendored here).
- Any **student lab report** or grading rubric—none are in this repository.

If a future revision of TAOS adds explicit `labs/` or `docs/hw*.md` files, this overview should be regenerated from those paths.
