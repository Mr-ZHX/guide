# YatCC — Labs Overview (English)

**YatCC** (*Yat Compiler Course*) is a **compiler construction** lab framework (中山大学编译课程 / related teaching). This document lists **only the labs that exist in the repository layout** (`task/0` … `task/5`, matching `test/task0` … `test/task5` and root `README.md`).

It states **what each task is for** and **what the toolchain must produce or pass**. It does **not** explain how to implement passes. It does **not** copy student reports or marketing text about external platforms.

**Build / grading:** CMake targets such as `taskN`, `taskN-score`, `taskN-pack` (see `task/0/README.md`). Optional **`config.cmake`** selects toolchain variants (e.g. **Flex+Bison** vs **ANTLR**) and whether **“revive”** mode feeds **reference outputs** from a previous task instead of your own.

---

## Task 0 — Environment setup

**Purpose:** Verify the **toolchain** (CMake, dependencies, LLVM/ANTLR setup as per project scripts) works end-to-end.

**What you do:** Build **`task0`**, run **`task0-score`**, then **`task0-pack`** for submission packaging. **Do not change** files under `task/0/` (doing so may void the score).

**Outcome:** A successful build and a reported score for the smoke test, proving the environment is ready for later tasks.

---

## Task 1 — Lexical analysis

**Purpose:** Implement a **lexer** for the course’s **SysY**-style C subset so that **preprocessed source** is turned into a **token stream** in the same style as the **clang** reference (token **kind**, **spelling**, **flags** such as start-of-line / leading space, and **source location**).

**Framework:** Two supported paths: **Flex** and **ANTLR** (each has its own subdirectory and local README describing where to edit). `config.cmake` chooses which path the grader uses.

**Outcome:** Token output matches the reference on the **task1** test suite.

---

## Task 2 — Syntax analysis (parsing)

**Purpose:** Complete **parsing** so the compiler builds the **abstract semantic graph (ASG)** and emits **clang-compatible syntax tree JSON**.

**Inputs (per official task text):**

- **Revive on:** token stream from **Task 1 reference** output.  
- **Revive off:** **Task 0** reference = **clang-preprocessed** source for each testcase.

**Output:** **JSON** representation of the syntax tree (as clang would structure it).

**Framework:** ASG definition, most parsing logic for **Bison** and **ANTLR**, and **ASG → JSON** serialization are largely provided; you fill in remaining parser / semantic actions (order-of-magnitude **~1k–2k LOC** mentioned in the handout).

**Outcome:** Parsed JSON matches the reference on the **task2** tests.

---

## Task 3 — Intermediate code generation (LLVM IR)

**Purpose:** Lower the ASG (from parsing) to **LLVM IR** (`.ll` text) that is **semantically equivalent** to **`clang -S -emit-llvm`** on the same source (your IR may differ syntactically).

**Inputs:**

- **Revive on:** **Task 2** reference **JSON** syntax tree.  
- **Revive off:** **Task 0** preprocessed source.

**Output:** **LLVM IR** files.

**Framework:** JSON → ASG loading is provided; the intended focus is **`EmitIR.hpp` / `EmitIR.cpp`**, though the handout allows fixing other files if needed. The grader compares behavior by compiling and **running** generated executables (artifacts such as `answer.ll` / `output.ll`, `.exe`, `.out` / `.err` are described in `task/3/README.md`).

**Outcome:** Correct programs on **functional** tests; semantics must match the reference (including defined behaviors such as **short-circuit** evaluation of logical operators).

---

## Task 4 — LLVM IR optimization

**Purpose:** Implement **optimization passes** on **LLVM IR** so output remains **semantically identical** to the input but **runs faster** (performance-scored).

**Inputs:**

- **Revive on:** **Task 3** reference **LLVM IR** (clang **-O0** style).  
- **Revive off:** **Task 0** preprocessed source (full pipeline from your front/middle stages if not revived).

**Output:** Optimized **LLVM IR**.

**Framework:** Demonstrates **LLVM AnalysisPass** and **TransformPass** usage. Included examples: **`StaticCallCounter`** (analysis) and **`StaticCallCounterPrinter`**, plus **`ConstantFolding`** (transform). You extend this with your own passes.

**Outcome:** Correctness preserved; **higher performance** on designated **performance** benchmarks yields better scores.

---

## Task 5 — Back-end code generation

**Official `task/5/README.md` in this tree** currently says the write-up is **to be updated**; treat detailed prose there as **pending**.

**What the framework code indicates:** A **full-pipeline skeleton** that (in `main.cpp`) drives **ANTLR** lexer/parser for **SysY2022**, **`MyVisitor`** to build an **internal middle-end IR**, optional **optimization** hooks (`visitor.opt()`), **printing**, optional **`TASK5_LLM`** integration for IR-related assistance, and **ARM-oriented back end** (`arm.*`, `backEnd/asm/*` assembly peephole / RA-style passes). Under `task/5/opt/` there are many named optimization modules (e.g. **DCE**, **mem2reg**, **LICM**, **inliner**, …) consistent with a **codegen + IR/lowering** capstone.

**Purpose (from root `README.md` only):** **Back-end code generation**—turning intermediate representation into **target assembly** (here, **ARM-family** tooling in the tree) with room for **machine-level** improvements.

**Outcome (expected when the task is active):** Assembled / linked behavior matches the specification on **task5** tests; exact rubric follows the published task page when it replaces the placeholder README.

---

## Summary (repository tasks only)

| Task | Theme | Primary deliverable |
|------|--------|---------------------|
| **0** | Environment | Build, score, pack without editing `task/0` |
| **1** | Lexer | Clang-style token stream |
| **2** | Parser | Clang-style syntax tree JSON |
| **3** | IR gen | Semantically correct LLVM IR |
| **4** | IR opt | Faster LLVM IR, same semantics |
| **5** | Back end | ARM (etc.) codegen framework; full spec when README updated |

---

*Artifact names like `test/cases/functional-*`, `performance`, `llm-backend` are test buckets under `test/`; they define coverage but are not separate “labs.”*
