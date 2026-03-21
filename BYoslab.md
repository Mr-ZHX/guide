# `easy_lab` — User-Level Threads Lab Overview (English)

This repository is a **single** hands-on assignment from the **Operating Systems** track at **BUPT** (北京邮电大学): implement a **minimal user-level threading (“stackful coroutine”)** library on **Linux x86-64**, with **assembly-assisted context switching**. The handout in `README.md` also teaches background on **registers**, **call stacks**, and the **System V x86-64 ABI**; this overview states **what you must deliver**, not how to code it.

**Deadline / contact** appear in `README.md` (course-specific). **Arm64** hosts are discouraged for this lab; use **x86-64** (native Linux, WSL, VM, or the provided **Dev Container**).

---

## What this lab is about

**Goal:** Build a **non-preemptive**, **FIFO** user-mode **scheduler** that multiplexes several **uthreads** on one OS thread, saving and restoring **CPU register context** so each uthread can resume where it **yielded**.

**Provided pieces:** Skeleton C code (`uthread.c`, `uthread.h`), a **context-switch** routine in **`switch.S`**, and small **test programs**. You complete the missing logic so the **API** below behaves as specified.

**Core API (behavioral contract):**

| Function | Role |
|----------|------|
| `init_uthreads` | One-time setup of the threading system. |
| `uthread_create(func, arg, name)` | Allocate a uthread, prepare its stack and entry so execution will eventually run `func(arg)`, and **enqueue** it for scheduling. |
| `schedule` | **Main** calls this to enter the **scheduler loop** and run queued uthreads until none remain (exact termination semantics follow the tests). |
| `uthread_yield` | A running uthread **voluntarily** returns control to the scheduler while preserving its continuation point. |
| `uthread_resume` | The scheduler **resumes** a suspended uthread at its saved point. |
| `thread_destroy` | Reclaim bookkeeping when a uthread’s entry function **finishes**. |

**Execution model (as stated in the handout):** Every uthread first enters a common trampoline **`_uthread_entry`** before calling the user function. Scheduling order is **FIFO**. The implementation uses the supplied **`thread_switch`** (assembly) to swap **integer register** contexts captured in **`struct context`**.

---

## Required deliverable — “base” tests

You must pass the three **automated tests** built with the default `Makefile`:

| Program | Purpose (high level) |
|---------|----------------------|
| **`simple`** | Sanity-check basic **create → schedule → run → finish** behavior. |
| **`pingpong`** | **Cooperative** handoff between multiple uthreads (yielding back and forth). |
| **`recursion`** | Uthreads that **recurse** deeply enough to stress **stack** setup and context integrity. |

**How to verify (local):** `make` then `make tests` (runs the three binaries in sequence).

**Grading note (from the handout):** the **online grader** is described as running **`make tests`** on your submission; **challenge** targets are **not** part of that automated score.

---

## Optional — `demo`

**`demo`** is a small companion binary to help you understand **how `thread_switch` is used**; it is **not** listed in the default `tests` target. Treat it as a **learning aid**, not a graded requirement.

---

## Optional — `metrics` (measurement)

**`metrics.c`** benchmarks **rough context-switch cost** when linked with your uthread implementation. **Build** target: `make metrics`. The course encourages discussing results and optimizations in **your own** report; the platform may **not** run `metrics` for grading.

---

## Optional — Challenge extensions

These are **extra** problems; **passing them is not required** for the base automated grading.

| Target | Theme |
|--------|--------|
| **`challenge1`** | Extend context handling beyond **integer** registers so **floating-point / SIMD** state is preserved (FPU-related tests). **Success:** output includes a line such as **`[PASS] FPU Mode Test Completed.`** |
| **`challenge2`** | Move from a **1 kernel thread : N uthreads** model toward **M : N** scheduling (multiple OS threads running uthreads) while keeping **atomicity / correctness** under the test’s expectations. **Success:** output includes **`[PASS] M:N Scheduling Test (Atomic Correctness)`**. |
| **`challenge3`** | Add **preemption** (time-sliced or interrupt-driven) so uthreads do not rely solely on **yield**. **Success:** output includes **`[PASS] Preemption Test (All threads finished)`**. |
| **Further idea (handout)** | On top of preemption, **synchronization** primitives (e.g. a **channel**-like pipe). |

**Caution:** The README warns that **challenge** code paths can **interfere** with base tests—keep them isolated or behind clean build flags if you experiment.

---

## Course submission expectations (non-technical)

The syllabus in `README.md` asks for a **code archive** (layout rules such as a top-level `lab2/` folder) plus a **short written report** (implementation summary, difficulties, reflection; optional challenge/metrics discussion). **Proportions** (e.g. report vs. code weight) are **course policy**, not repeated here as a “sample report.”

---

## Summary

| Item | Required? |
|------|-----------|
| Working uthread library + FIFO scheduler + `make tests` pass | **Yes** (core lab) |
| `demo` | No (learning) |
| `metrics` | No (measurement / report material) |
| `challenge1`–`challenge3` | No (optional extensions) |

---

*This document summarizes goals and artifacts only; it intentionally omits implementation hints from `README.md` (stack alignment, GDB, etc.).*
