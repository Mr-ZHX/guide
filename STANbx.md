# Assignment 1 (asst1) — Lab Framework Overview

This document summarizes **only what exists in this repository** (six programs under `prog1_*` … `prog6_*`, plus shared code in `common/`). It is meant for someone who has **not** read the course handout: what each part is about, what you are expected to **achieve**, and what **constraints** apply. It intentionally **does not** explain *how* to implement solutions.

**Source of truth in-tree:** `README.md`, `README_aarch64.md`, and the program directories.

---

## Assignment-level context

**Theme:** Parallel performance on a modern multi-core CPU with SIMD.

You work with two main ideas:

1. **SIMD:** many data elements processed in parallel *within* one core.
2. **Multi-core / multi-threaded parallelism:** independent work running on multiple cores (including effects of hyper-threading where applicable).

The assignment emphasizes **measuring and interpreting** speedup and utilization, not only writing code.

**Tooling note (from README):** Several programs use the **ISPC** compiler (`*.ispc` sources). Program 2 uses a course-specific **simulated vector intrinsic** layer (`CS149intrin.*`) rather than raw hardware intrinsics.

---

## Program 1 — `prog1_mandelbrot_threads/`

**What it is:** A Mandelbrot-set image generator. A sequential reference fills a pixel buffer; you parallelize the same computation using **C++ `std::thread`**.

**What you implement:** Multi-threaded Mandelbrot rendering that remains **correct** relative to the serial routine, with a work partition that scales from a small number of threads up through **8** (and you also consider **16** threads for comparison).

**What you analyze / deliver (conceptually):** Speedup vs. the serial baseline for at least **view 1**, graphs across thread counts, per-thread timing to explain load imbalance, and an improved static work mapping that yields strong speedup on **both** views **without** inter-thread synchronization. The write-up discusses why speedup is or is not linear and what hyper-threading implies.

**Artifacts:** Builds to a `mandelbrot` binary; writes PPM images (e.g. `mandelbrot-serial.ppm`). View selection is via command-line options described in `README.md`.

---

## Program 2 — `prog2_vecintrin/`

**What it is:** A small numeric kernel (`clampedExp`) that must be expressed using the **fake vector ISA** in `CS149intrin.h`, which models masked SIMD lanes and reports **vector instruction counts** and **lane utilization**.

**What you implement:** A vectorized `clampedExp` equivalent to the serial reference, valid for **arbitrary** array length `N` and **configurable** `VECTOR_WIDTH` (changed in the header).

**What you analyze / deliver (conceptually):** Correctness checks from the driver; utilization statistics when sweeping vector width (e.g. 2, 4, 8, 16) on a fixed test size. Optional extra credit: vectorized `arraySum` with stricter assumptions on `N` and `VECTOR_WIDTH`.

**Artifacts:** Builds to `myexp`; supports flags for logging and small test sizes (`README.md`).

---

## Program 3 — `prog3_mandelbrot_ispc/`

**What it is:** The same Mandelbrot workload as Program 1, but the parallel SIMD path is written in **ISPC** (`mandelbrot.ispc`) and called from C++.

**Part A — ISPC / `foreach` (single-core SIMD gang):** Understand how ISPC maps **program instances** to SIMD; run the ISPC Mandelbrot and relate observed speedup to hardware width (e.g. 8-wide AVX2) and **data-dependent** per-pixel cost (SIMD divergence).

**Part B — ISPC tasks (multi-core):** A variant launches **ISPC tasks** to use multiple cores. You adjust **only** the task launch configuration in `mandelbrot_ispc_withtasks()` to reach a large speedup over serial (target stated in README). Optional extra credit compares **threads** vs. **ISPC tasks** at scale.

**Artifacts:** `mandelbrot` binary with modes for tasks vs. non-tasks (`README.md`).

---

## Program 4 — `prog4_sqrt/`

**What it is:** ISPC code that applies an **iterative Newton-style `sqrt`** to a large array of inputs (default distribution given in starter). Serial, ISPC without tasks, and ISPC with tasks variants are compared.

**What you do:** Report speedups and **attribute** them to SIMD vs. multi-core. Then **design input arrays** that **maximize** and **minimize** ISPC (no-tasks) speedup over serial, and explain the results in terms of control flow / convergence behavior. Optional extra credit: a hand-written **AVX2 intrinsic** version competitive with ISPC.

**Artifacts:** `sqrt` binary; ISPC built with AVX2 target as noted in README.

---

## Program 5 — `prog5_saxpy/`

**What it is:** A **BLAS-style `saxpy`**: `result = scale * X + Y` over very large vectors (`N` on the order of tens of millions). Serial, ISPC (gang), and ISPC-with-tasks implementations are provided for study.

**What you do:** Measure reported performance; interpret speedup from tasks; argue whether **near-linear** speedup is realistic for this kernel and why (memory vs. compute). Optional extras: explain the stated **byte traffic** formula, or pursue a **significant** manual performance improvement.

**Artifacts:** `saxpy` binary; `saxpy.ispc` + `saxpySerial.cpp`.

---

## Program 6 — `prog6_kmeans/`

**What it is:** A **correct but slow** multi-threaded **K-means** on a large dataset (external `data.dat`; plotting via `plot.py` and `requirements.txt`).

**What you implement:** Speed up the program by changing **only** `kmeansThread.cpp`, **without** changing correctness (convergence, distance semantics, and visual sanity checks from the plots). You may parallelize **at most one** of: `dist`, `computeAssignments`, `computeCentroids`, `computeCost`. You must **not** alter `stoppingConditionMet` or the `kMeansThread` interface.

**What you do:** Use timing (e.g. `CycleTimer` from `common/`) to find hotspots, then optimize toward a **minimum speedup ratio** stated in the README. The write-up is expected to read as a measured, iterative performance investigation.

**Artifacts:** `kmeans` binary, logs for plotting, Python plotting script.

---

## Shared infrastructure — `common/`

**Role:** Utilities reused by programs (e.g. high-resolution timing in `CycleTimer.h`, task system code in `tasksys.cpp`, PPM I/O in `ppm.cpp`). Not a separate “lab,” but support for several exercises.

---

## Optional platform note — `README_aarch64.md`

**What it is:** Guidance for running or reporting results on **ARM (Apple Silicon)** machines. It is **not** a graded program directory; it is an alternate-environment appendix for curiosity / optional reporting (submission expectations remain per README).

---

## What this overview omits

- **Build commands, flags, Gradescope file names, and myth-machine setup** — see `README.md`.
- **Hints, algorithms, and implementation strategies** — intentionally excluded so this stays a neutral “what & why,” not a solution guide.
