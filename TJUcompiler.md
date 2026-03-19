# Toy Compiler (Rust-like -> RV32IM) — English Lab Manual

This English lab manual documents the **Toy Compiler** project located at `toy-compiler/`.
It describes the project goals, language design, compilation pipeline, key implementation modules, and how to build/run and validate the compiler.

Primary references in this repository:

- `README.md` (project overview and file structure)
- `report/design_report.md` (full system design: lexer/parser/semantic/IR/codegen)
- `note/production.md` (grammar/productions)
- `note/riscv.md` and `note/pseudo_instruction.md` (RISC-V backend notes)
- Source code under `src/` (entry, IR builder, code generation, register allocation)
- `test/*.rs` (sample programs that the compiler supports)

---

## 1. Project Overview

The Toy Compiler is a **Rust-like language compiler** implemented in modern C++.
Given a source program written in the project’s Rust-like syntax, the compiler can:

1. Generate an **intermediate representation (IR)** in a *quadruple-like* form.
2. Generate **RISC-V RV32IM assembly** as target code.

The generated assembly is designed to be linkable and runnable in a RISC-V Linux ABI–like environment under **QEMU** (see `riscv-asm/`).

### Supported Frontend/Backend Coverage (from `README.md`)

- The frontend (preprocess/lexer/parser/semantic/AST/IR builder) supports essentially all productions except the “borrow & reference” ones listed in the README.
- The backend (assembly generation) supports most productions, but explicitly excludes productions in the range **8.1 ~ 9.2** (as stated in `README.md` and the provided design docs).

---

## 2. Language Summary

The input language is a simplified, Rust-inspired language with:

- Function declarations: `fn <ID>(...) -> <Type>? { ... }`
- Statements modeled as expressions: blocks and control structures can appear in expression positions
- Variable declarations: `let mut x = Expr;` (type optional in some cases)
- Arithmetic and comparisons: `+ - * /`, and `== != < <= > >=`
- Control flow:
  - `if { ... } else { ... }` (also supports `else if`)
  - `while <expr> { ... }`
  - `for <mut var> in <expr>.. <expr> { ... }`
  - `loop { break <expr>?; continue?; }`
  - `break` and `continue`
- Basic data:
  - `i32`, `bool`, `()` unit type
  - Arrays `[T; N]` and array literals
  - Tuples `(T1, T2, ...)` and tuple element access like `.0` (and `.1`, etc.)

The formal grammar is described in `note/production.md`.

### Example Programs

Sample test programs are under `test/`.
For instance, `test/all_in_one.rs` demonstrates `if`, `while`, `for`, `loop`, `break`, and expressions returning values.

---

## 3. Repository Structure (Key Directories)

From `README.md`, the key directories are:

- `src/`
  - `preproc/` preprocess module (comment/annotation removal)
  - `lexer/` lexical analysis
  - `parser/` recursive descent parser (LL(2) style)
  - `semantic/` semantic checker
  - `symtab/` symbol table
  - `ir/` IR quad generation + IR builder
  - `codegen/` RISC-V assembly generation
  - `compiler/` compiler driver (orchestrates the pipeline)
- `report/` design report (the big design document)
- `note/` grammar and backend reference notes
- `riscv-asm/` runtime and QEMU make targets for executing generated assembly
- `test/` input programs in `.rs` format

---

## 4. Compilation Pipeline (End-to-End)

The pipeline is implemented by the compiler driver:

1. **Read input file** and keep its raw content for error reporting.
2. **Preprocess**: remove annotations/comments (`preproc::removeAnnotations`).
3. **Lexing**: produce tokens using `lex::Lexer`.
4. **Parsing + AST building**:
   - recursive descent parser constructs AST nodes
   - semantic checks and IR building are integrated (as described in the design report)
5. **Generate IR**:
   - `Compiler::generateIR()` writes `*.ir` (pretty printed if requested)
6. **Generate assembly**:
   - `Compiler::generateAssemble()` uses `cg::CodeGenerator` to emit `*.s`

### Entry Point / CLI Options

The executable entry in `src/main.cpp` provides:

- `-i, --input filename` : input file (with suffix)
- `-o, --output filename` : output base name (without suffix)
- `--ir` : generate IR only
- `--asm` : generate RISC-V assembly only

Example commands:

```bash
./toy_compiler --ir -i test.txt -o output
./toy_compiler --asm -i test.txt -o output
```

---

## 5. Key Implementation Modules

### 5.1 Preprocess

The preprocess stage removes annotations/comments so that the lexer/parser only sees the core language.

### 5.2 Lexer

Located in `src/lexer/`.
Core responsibilities:

- maintain a keyword table
- generate tokens with `(type, value, position)`
- recognize:
  - identifiers
  - integer literals
  - operators and delimiters

Design notes for the lexer are described in `report/design_report.md` (token structures and DFA-like scanning).

### 5.3 Parser (LL(2) recursive descent)

Located in `src/parser/`.

Important design points:

- The parser uses a small lookahead buffer (a “sandwich” model with `cur` and `la`), enabling LL(2) parsing without backtracking.
- Expressions are parsed with precedence layers (comparison -> add/sub -> mul/div -> factors).
- Left recursion is handled by iterative “while” expansion.
- The parser supports the “everything is an expression” language design (blocks and control flow returning values).

### 5.4 Semantic Checker + Symbol Table

Located in `src/semantic/` and `src/symtab/`.

Semantic checking guarantees:

- undeclared variables / functions are rejected
- variables must be initialized before use (unless formal parameters)
- function call arity matches
- return statement correctness matches the function’s declared return type
- scope rules are respected (nested blocks with shadowing)

The design and key checking procedures are described in `report/design_report.md`.

### 5.5 IR Builder (Quadruples)

Located in `src/ir/`.

Main behavior:

- `IRBuilder` traverses AST nodes and produces a **flat vector of IR quads** per function/program.
- The builder handles desugaring rules such as:
  - implicit returns for expression blocks/functions
  - propagation of the last expression value as a block expression result
- IR quads are created through factories like `QuadFactory`.

### 5.6 Code Generation (RISC-V RV32IM Assembly)

Located in `src/codegen/`.

Core behavior:

- Iterate over generated IR quads and emit corresponding assembly patterns.
- `CodeGenerator::generateFunc()` emits a `.global` label, resets allocators, allocates stack frame resources, then emits IR operations.
- Supported IR ops are handled in a switch statement (binary ops, assignment, goto/branches, call, label, return, etc.).

#### Register Allocation

Located in `src/codegen/reg_alloc.cpp`.

Design behavior:

- Prefer caller-saved registers; if exhausted, use callee-saved registers; if still exhausted, **spill** a register to the stack.
- It keeps per-register symbol pools and writes back values when required.
- Caller-saved registers are spilled before function calls; callee-saved are restored as needed.

---

## 6. Backend Reference: ABI and Pseudo Instructions

Backend notes:

- `note/riscv.md` explains:
  - RV32I core ISA assumptions
  - calling convention phases
  - register classes (caller/callee saved)
  - branch instructions usage
- `note/pseudo_instruction.md` documents the expansion mapping of pseudo-instructions to real RISC-V instructions (e.g., `ret`, `j`, `call`, `tail`).

These notes explain how the code generator’s emitted assembly is expected to behave.

---

## 7. Building and Running

### 7.1 Build the Compiler

In `toy-compiler/`:

```bash
make all
```

The root `Makefile`:

- compiles all `src/**/*.cpp` with `clang++` using `-std=c++26`
- generates `src/ast/accept.cpp` from `src/generate_accept.pl` using `perl`
- outputs a binary named `toy_compiler` in the build directory

Optional:

- `make verbose` to enable verbose build
- `make clean` to remove build artifacts

### 7.2 Generate Assembly

Example:

```bash
./toy_compiler --asm -i test/all_in_one.rs -o output
```

This produces `output.s` in the current directory (or as per the program output convention).

### 7.3 Run Under QEMU (RISC-V)

The directory `riscv-asm/` contains:

- `main.c` which calls `main0()` and prints the return value
- `Makefile` to build a static ELF with the generated `output.s` and run it via QEMU

Workflow:

1. Copy/prepare the generated assembly as `riscv-asm/output.s` (default `SRC ?= output.s`).
2. In `riscv-asm/`, run:

```bash
make qemu
```

For debugging:

```bash
make qemu-gdb
```

You may need a riscv64 toolchain installed on the host, matching the `TOOLPREFIX` logic used in the Makefile.

---

## 8. Testing and Validation

Validation methods:

1. **IR inspection**:
   - run with `--ir` to check that semantic desugaring and control-flow translation are correct
2. **Assembly correctness**:
   - run `--asm` and execute via `riscv-asm/` + QEMU
3. **Coverage via `test/*.rs`**:
   - each test file targets features like:
     - conditions, loops, comma operator variants, redefine behavior
     - `uninitialized` checks
     - `break`/`continue` correctness

If a run fails, a good strategy is:

- confirm semantic checker errors first (if compilation stops)
- then compare the IR output (should reflect the AST and semantics)
- then inspect emitted assembly around:
  - function prologue/epilogue
  - branch targets/labels
  - register spills/restores around calls

---

## 9. Deliverables (What This Lab Asks You to Demonstrate)

At completion, you should be able to:

- compile the project successfully with `make`
- compile at least a subset of programs from `test/`
- generate and verify:
  - IR output (`--ir`)
  - assembly output (`--asm`)
- run the produced executable under QEMU using the `riscv-asm` Makefile

---

## 10. Notes and Known Constraints

- The backend does not necessarily implement every production in the grammar (see `README.md`).
- Borrow/references features are restricted according to the project’s frontend coverage statement.
- If your host lacks a riscv64 cross toolchain, you may only be able to generate `.ir` and `.s` (no QEMU execution).

---

End of document.

