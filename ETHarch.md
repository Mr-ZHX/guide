# ETH Computer Architecture (`ETHarch`) — Lab Framework Overview (English)

This document summarizes **only what exists in this `ETHarch/` workspace**: three top-level lab folders (`lab1-problem`, `lab2_problem_ca_fall2022`, `lab3-template_pim_ca_fall2022`). Source headers attribute **Lab 2** to **ETH Computer Architecture** (Professor **Onur Mutlu**) and name **Fall 2022** in directory names; it is **not** an official course handout PDF.

There is **no root `README.md`** in this clone. This overview does **not** reproduce student reports or solution write-ups.

---

## Repository map

| Directory | Contents in this snapshot |
|-----------|---------------------------|
| **`lab1-problem/`** | **Empty** (no files checked in). The folder name suggests a **Lab 1** assignment slot, but there is **no code or spec** here to describe. |
| **`lab2_problem_ca_fall2022/`** | **MIPS-32 pipelined CPU simulator** in C: instruction-level simulator core (`shell.c`), **5-stage pipeline** logic (`pipe.c` / `pipe.h`), MIPS helpers (`mips.h`), plus a **Python test driver** (`run.py`) and large **`inputs/`** trace libraries (`.x` / `.s` pairs). |
| **`lab3-template_pim_ca_fall2022/`** | **Processing-in-memory (PIM) / UPMEM DPU** template: **host** code (`host/app.c`), **DPU** kernels (`dpu/task.c`), shared headers (`support/`), **Makefiles**, and **Docker** scripts to run the UPMEM toolchain environment. |

---

## Lab 1 — `lab1-problem/` (no materials in tree)

**Status:** The directory exists but contains **no source files** in this repository.  

**Implication:** Nothing can be inferred about tasks or deliverables from the framework alone. If your course uses a separate Lab 1 handout or zip, it is **not** vendored here.

---

## Lab 2 — `lab2_problem_ca_fall2022/` (MIPS pipeline simulator)

**Goal:** Build or complete a **cycle-accurate (or cycle-oriented) 5-stage MIPS-32 pipeline simulator** that matches a **reference “baseline” simulator** (`basesim` in `run.py`—your build is expected as `./sim`).

**What the framework provides:**

- **`shell.c`** — **Memory image**, register file, **non-pipelined** or baseline execution path, and statistics (`stat_cycles`, etc.). Marked **do not modify** in the banner (students work elsewhere).
- **`pipe.c` / `pipe.h`** — **Pipeline state machine**: stages `fetch → decode → execute → mem → writeback`, branch recovery / flushing, and per-instruction `Pipe_Op` structures.
- **`run.py`** — Runs many `.x` input programs, compares **register / statistics** output between **reference** and **your** simulator, and reports **REGISTER CONTENTS OK** or errors.
- **`inputs/`** — Organized test suites (**`short`**, **`medium`**, **`long`**, **`random`**, etc.) covering arithmetic, branches, memory, jumps, and long stress sequences.

**What you implement (conceptually):** Correct **pipeline behavior** for the MIPS subset the course defines (hazards, stalls, forwarding, branch resolution—**exact requirements are in the external assignment**, not in this repo).

**Deliverable:** A compiled simulator binary (typically `make` → `sim`) that passes **`run.py`** against the provided inputs.

---

## Lab 3 — `lab3-template_pim_ca_fall2022/` (UPMEM DPU / PIM lab)

**Goal:** Program **UPMEM DPUs** (DRAM-processing “memory-side” accelerators) using the **UPMEM SDK** model: a **host** program allocates DPUs, loads a **DPU binary**, moves data, and collects results; **tasklets** on each DPU run **kernels** (often **vector** / **AXPY**-style workloads).

**What the framework contains:**

- **`template/task1/`** and **`template/task2/`** — Parallel **task** trees; each has **`host/`** (CPU-side `app.c` using `dpu_alloc`, `dpu_load`, `dpu_launch`, etc.) and **`dpu/`** (`task.c` with `main_kernel1`, barriers, **MRAM** / heap helpers).
- **`task2/dpu/task.c`** includes a **placeholder** comment **`//@@ INSERT AXPY CODE`** inside a local **AXPY** helper—students are expected to **fill** the computation for the cached block.
- **`support/`** — Shared types, timers, optional **cycle/instruction** counters (`cyclecount.h` in task2), parameter parsing.
- **`docker/`** — **`Dockerfile`** and scripts (`start_docker.sh` / `.bat`) to obtain a **reproducible** build environment for the DPU toolchain.

**What you implement (conceptually):** Correct **data movement** between host and MRAM, **multi-tasklet** coordination (barriers), and **high-performance** AXPY (or related) kernels on the DPU, with optional **performance measurement** (cycles/instructions as wired in the template).

**Deliverable:** Host + DPU binaries built via the provided **Makefiles**, runnable under the **Docker** / SDK setup described by the course.

---

## Cross-cutting notes

- **No autograder** is visible for Lab 3 in this tree beyond the **Makefile** / manual run workflow.
- **Lab numbering** follows folder names; **Lab 1** is missing content here.
- **Course branding** in comments (“Computer Architecture”, ETH-style naming) is **context only**; your syllabus may differ.

---

## What this overview omits

- **Pipeline hazard policies** and **exact MIPS opcode lists** for Lab 2.
- **UPMEM API details** and **grading rules** for Lab 3.
- **How to** implement AXPY, forwarding, or Docker setup—those belong in the official lab documents.

If you add `lab1-problem` files or a `README` later, regenerate this overview from those paths.
