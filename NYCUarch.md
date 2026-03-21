# Computer Organization 2024 (NYCU) — Labs Overview (English)

This repository is **2024 Spring** **Computer Organization** material from **National Yang Ming Chiao University (NYCU)** (**CS10014**). Official sites: [lecture](https://people.cs.nycu.edu.tw/~ttyeh/course/2024_Spring/CS10014/outline.html), [labs](https://nycu-caslab.github.io/CO2024/index.html).

This document summarizes **what each included lab is for** and **what you must deliver**. It does **not** teach implementation or copy student reports.

---

## What is actually in *this* repository

**Only three graded lab trees are present here:**

| Folder | Lab |
|--------|-----|
| `lab0/` | Lab 0 |
| `lab1/` | Lab 1 |
| `lab5/` | Lab 5 |

The **`COCPU/`** directory is an **optional appendix** (FPGA bitstream + bare-metal C) for running a small program on the course pipeline CPU; it is **not** the same as Lab 2–4 source handouts.

**Labs 2, 3, and 4** (branches on single-cycle, simple pipeline, advanced pipeline with hazards) appear on the [official lab site](https://nycu-caslab.github.io/CO2024/index.html) as part of the **full** course sequence, but **their starter code is not included in this checkout**. If you follow the full course, you normally **extend** the Lab 1 design on your own machine for those weeks.

---

## Lab 0 — Environment setup and basic Verilog

**Goal:** Set up **Verilator** + **GTKWave** and practice small Verilog blocks.

**What you do:**

- **Part 1:** **`fullAdder.v`** + C++ **testbench** (`lab0/part1/`).
- **Part 2:** **`alu.v`** + C++ testbench (`lab0/part2/`).

**Outcome:** You can `make`, simulate, and view waves; same flow as later CPU work.

---

## Lab 1 — Single-cycle CPU (simple RISC-V subset)

**Goal:** Implement a **single-cycle** CPU for a **minimal RV32I-style** subset.

**What you implement:** Modules under `lab1/` — e.g. **PC**, **adders**, **muxes**, **instruction memory** (from **`TEST_INSTRUCTIONS.txt`**), **register file**, **ImmGen**, **ALU** / **ALUCtrl**, **Control**, **data memory**, top **`SingleCycleCPU.v`**. Many files are **stubs** with `TODO` until you fill them in.

**What it must do:** Run **`TEST_INSTRUCTIONS`** and pass **`example_testbench.cpp`** (Verilator).

**Outcome:** Correct single-cycle execution for the ISA subset defined in the official Lab 1 handout.

---

## Lab 5 — Cache manager (C++ simulation)

**Goal:** Implement **`CacheManager`** on top of the provided simulator (`Memory`, `Cache`, `Evaluator`, `main`).

**What you implement:** Hit/miss logic, fill/evict, interaction with **backing memory** on miss, metadata (valid/tag/dirty, etc.) for **your** chosen geometry (**direct-mapped** or **set-associative**, replacement/write policy as allowed by the spec). **`process.py`** can build **`testcase.txt`** from **`Trace.txt`**.

**Outcome:** Build succeeds and the **evaluator** accepts your cache behavior (see `CacheManager.h` comments for constraints).

---

## Optional appendix — `COCPU/`

**Goal:** Run a **very small C program** on a **synthesized pipeline CPU** (e.g. Mimas A7 / Vivado project, **`PipelineCPU.v`**, UART / seven-segment debug, **`cpu_lib`**, **`hello.c`**).

**Note:** Intended for students who already have a **working Lab 4-class pipeline** from elsewhere; this repo **does not** ship Lab 2–4 Verilog sources.

---

## Summary (this repo only)

| Lab | Focus | Deliverable |
|-----|--------|-------------|
| **0** | Toolchain + Verilog | Full adder + ALU + waves |
| **1** | Single-cycle RISC-V subset | Completed `SingleCycleCPU` + Verilator tests |
| **5** | Cache simulation | Working `CacheManager` + evaluator |
| **COCPU** *(opt.)* | FPGA + bare C | Optional demo (needs pipeline CPU from full course) |

---

*For Labs 2–4 handouts and tests, use the [CO2024 lab website](https://nycu-caslab.github.io/CO2024/index.html). This file matches **this** tree: **lab0, lab1, lab5** only.*
