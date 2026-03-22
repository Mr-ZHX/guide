# CS61C-Assignment — Lab & Project Overview (English)

This document summarizes **what each lab and project in this repository appears to be about**, based only on **directory layout, filenames, and starter-code comments**—not on external course handouts or student write-ups. It is meant to help someone unfamiliar with the course **quickly understand the goals and deliverables at a high level**. It does **not** explain how to implement anything.

**Disclaimer:** This tree is an **unofficial** mirror of UC Berkeley CS61C–style assignments. For authoritative grading rules, deadlines, and full specifications, refer to the **official CS61C** materials for the semester you are following.

---

## Labs (`Lab/`)

### Lab 00

**What’s in the repo:** Text answer files only (`first_set.txt`, `answers.txt`).

**What it’s about (inferred):** A short **introductory / logistics** exercise—typically reading questions or simple written responses—rather than programming.

---

### Lab 01

**What’s in the repo:** C sources including `hello.c`, `interactive_hello.c`, `eccentric.c`, `ll_cycle.c` / `ll_cycle.h` / `test_ll_cycle.c`, `segfault_ex.c`, `no_segfault_ex.c`, and `gdb.txt`.

**What it’s about (inferred):** **C toolchain basics**, simple programs, **debugging with GDB**, and **linked-list** manipulation (cycle detection), plus examples that illustrate segmentation faults vs. correct behavior.

---

### Lab 02

**What’s in the repo:** C modules such as `bit_ops`, `lfsr`, and `vector`, with tests and a `Makefile`.

**What it’s about (inferred):** **Bit-level operations**, a **linear feedback shift register (LFSR)**-style exercise, and a **dynamic array (“vector”)** data structure in C—foundations for representing data at the machine level.

---

### Lab 03

**What’s in the repo:** RISC-V assembly files (e.g. `factorial.s`, `list_map.s`, `ex1.s`, `ex2.s`, `cc_test.s`) and a small C companion (`ex2.c`).

**What it’s about (inferred):** **RISC-V assembly programming**—calling conventions, control flow, and translating simple algorithms (e.g. factorial, list mapping) to assembly.

---

### Lab 04

**What’s in the repo:** Assembly files `megalistmanips.s` and `discrete_fn.s`.

**What it’s about (inferred):** More advanced **RISC-V assembly**—working with **structured list-style data** and implementing a **piecewise / discrete function** in assembly.

---

### Lab 05

**What’s in the repo:** Logisim circuits `ex1.circ`–`ex5.circ`, plus a `testing/` harness (Python script, reference outputs, test `.circ` files) and `test.sh`.

**What it’s about (inferred):** **Combinational / sequential logic in Logisim**—building and verifying small digital building blocks with automated tests.

---

### Lab 06

**What’s in the repo:** `ex1.circ` and a `ROMdata` file.

**What it’s about (inferred):** **Logisim ROM / finite-state style design**—using preloaded ROM contents as part of a circuit (often tied to control or lookup tables).

---

### Lab 07

**What’s in the repo:** C code for `transpose.c`, `matrixMultiply.c`, tests, `Makefile`, RISC-V `cache.s`, and text files `exercise1.txt`–`exercise3.txt`.

**What it’s about (inferred):** **Caches and memory hierarchy** concepts—matrix operations as workload drivers, **cache-friendly coding**, and written exercises; the assembly file suggests tying behavior to **ISA-level** or **microarchitectural** ideas.

---

### Lab 08

**What’s in the repo:** `answers.txt` only.

**What it’s about (inferred):** A **written / short-answer** lab (no code in this snapshot)—often conceptual questions rather than programming.

---

### Lab 09

**What’s in the repo:** `simd.c`, `simd.h`, `test_simd.c`, `Makefile`.

**What it’s about (inferred):** **SIMD** (single-instruction multiple-data) programming—vectorizing numeric kernels using compiler intrinsics or similar APIs, validated by unit tests.

---

### Lab 10

**What’s in the repo:** OpenMP-related sources (`omp_apps.c`, `omp_apps.h`, `v_add.c`, `dotp.c`), a small HTTP stack (`server.c`, `server_utils.*`, `libhttp/`), BMP helpers (`libbmp/`), static files under `files/`, `timer.sh`, and `Makefile`.

**What it’s about (inferred):** **Parallelism with OpenMP** (parallel loops, reductions) and a **minimal web server** that serves content—tying **performance measurement** (timing scripts) to **multi-threaded** code.

---

### Lab 11

**What’s in the repo:** Python scripts (`wordCount.py`, `perWordDocumentCount.py`, `mostPopular.py`, `createIndices.py`), Java (`textImporter/Importer.java`), sample text under `textFiles/`, `Makefile`, and a `submit` helper.

**What it’s about (inferred):** A **large-scale text processing** lab in the spirit of **MapReduce / Hadoop**—ingesting text, building indices, counting words per document, and finding popular terms across a corpus.

---

### `Lab/tools`

**What’s in the repo:** Shared tooling for labs (not summarized here as a separate “assignment”; see files inside for purpose).

---

## Projects (`Project/`)

### Project 1

**What’s in the repo:** C sources such as `gameoflife.c`, `steganography.c`, `imageloader.c`, `imageloadertester.c`, and extensive `testOutputs/` (including `.ppm` reference images).

**What it’s about (inferred):** **Image processing in C** using the **PPM** format—implementing **Conway’s Game of Life** on images and a **steganography** task that hides or recovers data in images. Starter comments in `gameoflife.c` describe wrapping boundaries and rule-based updates.

---

### Project 2

**What’s in the repo:** RISC-V assembly under `src/` including `matmul.s`, `dot.s`, `relu.s`, `argmax.s`, `read_matrix.s`, `write_matrix.s`, `classify.s`, `main.s`, and helpers; directories for `tests`, `tools`, etc.

**What it’s about (inferred):** A **RISC-V assembly** project that implements pieces of a **matrix-based classifier** (neural-net-style building blocks: dot products, matrix multiply, ReLU, argmax, I/O of matrices). The local `README.md` may contain informal notes; treat the **official project spec** as authoritative for exact requirements.

---

### Project 3

**What’s in the repo:** Logisim `.circ` CPU-related files and a rich `tests/` tree (`part_a` ALU/regfile-style checks; `part_b` **single-cycle** and **pipelined** CPU tests with assembly inputs and reference outputs).

**What it’s about (inferred):** **CPU design in Logisim**—register file, ALU, then a **single-cycle RISC-V CPU**, extended to a **pipelined** design with hazard handling; verification against reference execution traces.

---

### Project 4

**What’s in the repo:** C extension code (`numc.c`, `matrix.c`, headers), `Makefile`, Python `unittests/`, and scripts—typical of a **Python C extension** project.

**What it’s about (inferred):** **`numc`**-style **fast matrix operations** exposed to Python—implementing a `Matrix61c`-like type with correct semantics and **performance** competitive with optimized libraries, validated by Python unit tests.

---

## How to use this document

- Use it to **orient** yourself: what theme each folder corresponds to (C, assembly, Logisim, SIMD, parallelism, Hadoop-style jobs, CPU design, Python extensions).
- For **exact** point breakdowns, allowed libraries, and collaboration rules, always use the **course’s official** handouts and autograder expectations.
