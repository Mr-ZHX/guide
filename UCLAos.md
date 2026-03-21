# UCLA CS 111 (Winter 2017) — Lab Framework Overview (English)

This document summarizes **only what this repository’s directory layout and top-level `README.md` imply** about the CS 111 assignment sequence. It is meant for someone who has never taken the course: **what each lab is about** and **what kind of program or artifact you produce**. It **does not** explain how to implement solutions.

**Course framing (in-tree):** The root `README.md` identifies the offering as **UCLA CS 111, Winter 2017** (Paul Eggert) and links to the public lab pages for details, deadlines, and full specifications.

**Note on other `README` files:** Several subfolders contain **student submission READMEs** (file lists, testing notes, answered questions). Those are **not** official handouts; this overview **does not** reuse that Q&A text.

---

## Repository map (labs present here)

| Folder | Focus |
|--------|--------|
| `Lab 0/` | Warm-up C program |
| `Lab 1/Lab 1A`, `Lab 1B`, `Lab 1C` | Incremental milestones for **`simpsh`** (simple shell) |
| `Lab 2/Lab 2A` | Multithreading, races, synchronization, measurements |
| `Lab 2/Lab 2B` | Lock granularity, partitioning, profiling, throughput experiments |
| `Lab 3/Lab 3A` | EXT2 file-system image **dump** (C) |
| `Lab 3/Lab 3B` | File-system **analysis** from CSV outputs (Python) |
| `Lab 4/` | Embedded work on **Intel Edison** (sensors + networked client) |

Build tooling is **Make**-based in each lab; **GCC** is used for C. Lab 4 also links **libmraa** (hardware I/O), **pthread**, and (for one part) **OpenSSL** (`-lssl -lcrypto`), per `Lab 4/Makefile`.

---

## Lab 0 — Warm-up (`Lab 0/`)

**Goal (from course README):** A very small **“trivial”** exercise—on the order of a quick refresher—to practice **command-line parsing** and basic C workflow.

**What you produce:** A single C source module (`lab0.c`) plus `Makefile`. The starter behavior suggested by the code structure includes **long options** (e.g. input/output file redirection-style flags) and optional exercises involving **signals / segmentation faults** and **debugging with gdb** (as typically required on the course’s Project 0 page).

**Skills:** `getopt_long`, file descriptors, basic error handling, `make` targets (often including a **`check`** rule).

---

## Lab 1 — Simpleton Shell (`Lab 1/Lab 1A` … `Lab 1C`)

**Goal (from course README):** Implement **`simpsh`**, a **minimal shell** in C that is driven entirely by **command-line arguments** (not an interactive REPL). It must open the requested files, create any **pipes**, spawn **subprocesses** with the correct **stdin/stdout/stderr** wiring, and **report exit statuses** as children terminate.

**What you produce:** `simpsh.c` plus `Makefile`, developed in **three staged directories** (`Lab 1A`, `Lab 1B`, `Lab 1C`) representing **incremental feature completion** (typical CS 111 pattern: file open flags and `--command` FD mapping first; later phases add richer process/pipe orchestration per the official spec).

**Illustration of early behavior (from `Lab 1A/Makefile` tests):** Invocations combine options such as **`--rdonly`**, **`--wronly`**, and **`--command`** with numeric file-descriptor slots so a utility like `cat` copies data between files—sanity-checked with `diff`.

**Later milestone (mentioned in student metadata only as a deliverable type):** A **report** comparing performance or behavior of `simpsh` against common shells (e.g. Bash/Dash)—confirm exact requirements on the course **Lab 1** page.

---

## Lab 2A — Races and synchronization (`Lab 2/Lab 2A`)

**Goal (from course README):** **Multithreaded C** programs that expose **data races** on a shared counter and on a **sorted doubly linked list**, then correct them with **synchronization primitives** (mutex, spin-lock, etc.). You **measure** behavior (timing, failures) and **plot** results with **gnuplot**.

**What you produce:**

- `lab2_add.c` — threaded **add-to-shared-variable** experiment with CLI-controlled parameters and CSV output.
- `SortedList.h` (provided interface) + `SortedList.c` — thread-safe (or intentionally unsafe-with-yields) **sorted list** operations matching the supplied API.
- `lab2_list.c` — threaded **list workload** driver with similar measurement output.
- CSV result files, **gnuplot** scripts (`*.gp`), generated **PNG** graphs, and a `README.txt` documenting runs (the official spec defines which plots and questions are required).

**Conceptual outcomes:** Observe incorrect results under races, effect of `sched_yield`, and relative costs of synchronization choices.

---

## Lab 2B — Lock granularity and performance (`Lab 2/Lab 2B`)

**Goal (from course README):** Extend the Lab 2 thread/list theme to **performance analysis**: how **lock granularity**, **partitioned data structures**, and **contention** affect throughput and wait times; use **profiling** (e.g. `gperf`/`profile.gperf`-style reports in this tree) to see where CPU time goes.

**What you produce:** Updated or parallel drivers (`lab2_add.c`, `lab2_list.c`, `SortedList.*`), aggregated CSV (`lab_2b_list.csv` here), gnuplot script `lab2b.gp`, multiple **throughput / timing** plots (`lab2b_*.png`), profiling output, and documentation `README.txt` per assignment instructions.

---

## Lab 3A — File system dump (`Lab 3/Lab 3A`)

**Goal (from course README):** Read a **binary image** of an **ext2** file system, **parse on-disk structures**, and emit **summaries as CSV files** (superblock, groups, inodes, directory entries, indirect blocks, etc.—exact file list on the course Project 3A page).

**What you produce:** `lab3a.c` + `Makefile`. The program uses low-level read primitives (e.g. `pread`) over the image and writes multiple **`.csv`** outputs suitable for automated checking (`diff` against reference outputs).

---

## Lab 3B — File system analysis (`Lab 3/Lab 3B`)

**Goal (from course README):** A **Python** tool that ingests the **CSVs produced in 3A** and **diagnoses inconsistencies or corruption** in the file system, printing a structured report of detected problems.

**What you produce:** `lab3b.py` + `Makefile` (often with a `run` target). Execution assumes the CSVs from 3A are available in the working directory.

---

## Lab 4 — Embedded systems (`Lab 4/`)

**Goal (from course README):** C programs targeting **Intel Edison** hardware using **MRAA** to read a **Grove temperature sensor**, plus a **network client** that talks to a remote server over **TCP** or **SSL/TLS** (OpenSSL), depending on the part.

**What you produce:** Three sources—`part1.c`, `part2.c`, `part3.c`—built to binaries `lab4_1`, `lab4_2`, `lab4_3`, with corresponding **log files** (`lab4_*.log`) capturing required session output, plus `Makefile` and `README`.

**Libraries (from Makefile):** `mraa`, math (`m`), `pthread`, and OpenSSL/crypto for the SSL-capable binary.

---

## What this overview omits

- **Official due dates, grading weights, slip days, and partner rules** — only on the course website snapshots linked from `README.md`.
- **Exact CLI option lists, CSV column schemas, and Edison server protocols** — full specs are on the linked assignment pages.
- **Step-by-step implementation guidance.**

If your local tree differs from the 2017 pages (renamed options, updated boards), treat the **course handout for your term** as authoritative.
