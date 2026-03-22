# UCSD CSE 131 / Compilers (`ucsd_cse131`) — Lab Framework Overview (English)

This document summarizes **only what this repository’s assignment folders contain**: a **sequence of eight Rust (`cargo`) projects** that implement compilers for dialects of the course’s **S-expression “.snek” language**, lowering programs to **x86-64 assembly** (`.s` files) consumed by the course runtime. It is intended for readers unfamiliar with the course: **what each assignment adds** and **what the compiler must do**, without algorithms or solution strategies.

**External context (in-tree pointer only):** The root `README.md` links the public course site (`ucsd-compilers-s23.github.io`). Full grammars, deadlines, and runtime (`runc` / linking rules) live there; this repo is a **solution snapshot**, not the official handout PDFs.

**Note:** The root `README.md` contains informal claims about grades; this overview **does not** treat those as specifications.

---

## Common technical pattern

Across assignments you typically:

1. **Read** a source file (`.snek` / embedded in tests).
2. **Parse** S-expressions (the `sexp` crate appears throughout).
3. **Compile** an internal AST to **assembly** text.
4. **Write** a `.s` file whose entry label is usually `our_code_starts_here`, often with a **`snek_error`**-compatible error path once runtime checks exist.

Later assignments add **integration tests** under `tests/` (Rust `all_tests.rs` + `.snek` / golden `.s` pairs) and may use **`prettydiff`** for asm comparison.

---

## Assignment 1 — `assignment1-adder` (Adder)

**Working name:** `adder` (Cargo package).

**Language subset:** Tiny arithmetic expressions: numeric literals, **`add1`**, **`sub1`**, **`negate`** (nested as S-expressions).

**Deliverable behavior:** A compiler that reads an input path and output path, parses one expression, and emits a minimal **NASM-style** text section with **`our_code_starts_here`** returning the evaluated value in `rax`.

**Tests:** Under `test/`, paired `.snek` and expected `.s` files (e.g. add/subtract chains, negation).

---

## Assignment 2 — `assignment2-boa` (Boa)

**Language extensions over Adder:** **Identifiers**, **binary `+`, `-`, `*`**, and **`let` bindings** (multi-binding `let` as nested structure).

**Deliverable behavior:** Compile expressions with **variables** and **arithmetic** to assembly using an internal **IR-style instruction enum** (`IMov`, `IAdd`, etc.) and register/stack discipline appropriate to the starter design.

**Tests:** Large `tests/` corpus including binding and nested arithmetic examples.

---

## Assignment 3 — `assignment3-cobra` (Cobra)

**Language extensions over Boa:** **Booleans**, rich **comparisons**, **`if`**, imperative features **`block`**, **`set!`**, **`loop` / `break`**, and **overflow / invalid-argument** error reporting via fixed **error codes** (statically visible in sources).

**Runtime model:** Values move toward a **tagged representation** on the integer path (error constants and comparison/bitwise instruction forms appear in the compiler).

**Deliverable behavior:** Compile control flow and comparisons with correct **dynamic error** behavior per course ABI (calls into **`snek_error`** in later asm templates).

**Tests:** Many `.snek` cases including error and control-flow scenarios.

---

## Assignment 4 — `assignment4-Caduceus` (Caduceus)

**Structure:** A **multi-module** compiler (`parser`, `compiler`, `types`, `util`) rather than a single `main.rs` monolith. This folder contains **more than one Cargo workspace copy** (`compiler_09/`, `compiler_57/`)—functionally the same assignment scaffold in duplicate trees.

**Language features (from `parser` / `types`):** A richer **expression** language including **`block`**, **`set!`**, **`if`**, **`let`**, **booleans**, **numeric bounds checks at parse time**, and **`input`**. Compilation returns **structured `CompileError`s** instead of only panics.

**Deliverable behavior:** **Robust parse + compile** pipeline producing assembly that wraps `our_code_starts_here` with a **`throw_error`** stub that forwards to **`snek_error`**, matching the course’s error story.

---

## Assignment 5 — `assignment5-diamondback` (Diamondback)

**Major addition:** **First-class function definitions** (`fun` …) and **calls**—programs are a sequence of **definitions** plus a main expression (`Defn` + `Call` in the AST).

**Language:** Retains prior expressions (including loops, sets, blocks, booleans, comparisons) and adds **function parameters** and **call sites**.

**Deliverable behavior:** Implement **closures-free** (typically static linking style) function compilation: **calling conventions**, stack frames, and correct interaction with existing runtime error checks.

---

## Assignment 6 — `assignment6-EggEater` (Egg Eater)

**Major addition:** **Heap-allocated tuples** constructed from expressions, **indexed access** (`Index`), while keeping **functions**.

**Deliverable behavior:** Represent **tuple values** in memory with **tag/bounds** checking (distinct runtime error codes for bad tags vs. out-of-range indices appear in sources).

**Tests:** Tuple-heavy programs and tag/bounds error cases (see that assignment’s `tests/` folder).

---

## Assignment 7 — `assignment7-ForestFlame` (Forest Flame)

**Major shift:** **Garbage-collected vectors** instead of (or in addition to) simpler tuple models—AST includes **`make-vec` / vector literals**, **`vec-get` / `vec-set` / `vec-len`**, **`nil`**, **`gc`**, **`print-stack`**, **`input`**, and **integer division** in the operator set.

**Structure:** Clean separation into **`asm`**, **`compiler`**, **`parser`**, **`syntax`** modules; `main` is a thin driver.

**Deliverable behavior:** Compile a **memory-safe** vector API with a **tracing GC** interface (explicit `gc` node) per course ABI—allocation, pointer tagging, and collector hooks as specified on the course site.

---

## Assignment 8 — `assignment8-GreenSnake` (Green Snake)

**Extensions over Egg-style tuples:** **Structural equality** (`StructuralEqual` binary op), **tuple field update** (`SetTuple`), and the full combination of **functions**, **tuples**, **indexing**, and prior control flow.

**Deliverable behavior:** Complete the **largest dialect** in the sequence: compile all constructs with correct **equality semantics** (structural vs. pointer/value—per handout) and **mutation** of tuple slots.

**Tests:** Broad `tests/` suite including cycles, BST-style programs, duplicate-parameter errors, etc.

---

## What this overview omits

- **Exact syntax** of every S-expression form and **grading rubrics** — see the course website.
- **Linking command lines**, **reference outputs** for every test, and **runtime source** (not all vendored here).
- **Why** two `compiler_*` trees exist under Assignment 4 — treat as duplicate scaffolding unless your instructor says otherwise.

For authoritative specs, use the **GitHub course org** materials for your term (e.g. Spring 2023 site linked from `README.md`).
