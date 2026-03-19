# Compiler-Lab — SysY to Koopa IR and RISC-V (English Lab Manual)

**Course**: Compiler Principles (PKU minic / compiler-dev style labs)  
**Implementation**: Rust + `lalrpop` for parsing + `koopa` IR + RISC-V assembly generation

This document is a complete English lab manual for the `Compiler-Lab` project in this directory. It explains the overall goal, how to run the compiler, the implemented compilation pipeline, and the lab work progression **Lv.1 ~ Lv.9**.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Supported Features / Grammar Summary](#2-supported-features--grammar-summary)
3. [How to Build and Run](#3-how-to-build-and-run)
4. [Command-Line Interface](#4-command-line-interface)
5. [Compilation Pipeline](#5-compilation-pipeline)
6. [Lab Progression (Lv.1 ~ Lv.9)](#6-lab-progression-lv1--lv9)
7. [Testing with `autotest` (Docker)](#7-testing-with-autotest-docker)
8. [Project Code Map](#8-project-code-map)
9. [Known Limitations](#9-known-limitations)

---

## 1. Project Overview

### 1.1 Goal

The compiler takes a small **SysY**-like C subset (as defined by the parser grammar) and translates it into:

1. **Koopa IR** (also called “Koopa” in the project): generated in-memory and dumped as text by the compiler mode `-koopa`.
2. **RISC-V assembly**: generated from Koopa IR by the backend mode `-riscv`.

### 1.2 Modes

From the code in `src/main.rs`, the compiler supports the following modes:

- `-koopa`: parse input and output Koopa IR text to the output file.
- `-riscv`: parse input and output RISC-V assembly code to the output file.
- `-perf`: treated like `-riscv` in this implementation (assembly generation).

### 1.3 Implemented Lab Levels

The repository records the work in the original (Chinese) `README.md` as **Lv.1 to Lv.9**, covering:

- parser + AST
- basic IR generation
- expressions
- constants and variables
- blocks and scopes
- `if` / short-circuiting
- `while` / break and continue
- functions and global variables
- arrays

---

## 2. Supported Features / Grammar Summary

The grammar defined in `src/sysy.lalrpop` / `src/ast_def/` corresponds to a SysY subset with:

### 2.1 Program structure

- `CompUnit`: a list of units
  - `Decl` (const/var declarations)
  - `FuncDef` (function definitions)

### 2.2 Declarations

- Function definition:  
  `BType IDENT "(" [FuncFParams] ")" Block`
- Const declaration:
  `const BType ConstDef { "," ConstDef } ";"`  
  (array shapes supported; initial values are const-evaluable in semantic stage)
- Var declaration:
  `BType VarDef { "," VarDef } ";"`  
  (array shapes supported; initial values for globals are required to be const by semantics)

### 2.3 Statements

The AST/grammar supports:

- assignment: `LVal "=" Exp ";"`
- expression statement: `[Exp] ";"`
- blocks: `{ ... }`
- `if (Exp) ... [else ...]` (with ambiguity resolved in Lv.6)
- `while (Exp) ...`
- `break;`, `continue;`
- `return [Exp];`

### 2.4 Expressions

- LOrExp, LAndExp, equality, relational, add, mul, unary, primary
- unary operators: `+`, `-`, `!`
- lvalues:
  `IDENT {"[" Exp "]"}`
- literals:
  `INTCONST`

### 2.5 Short-circuit / boolean lowering

The implementation includes special handling for:

- logical `||` and `&&` using control-flow blocks and short-circuit semantics
- boolean lowering via comparisons and “!= 0” style conversions (Koopa IR lacks direct `and/or` for integers)

---

## 3. How to Build and Run

### 3.1 Docker / Container Environment (recommended)

The project’s README suggests running inside a prepared container:

```bash
docker run -it -v D:/MyCodes/Compiler-Lab:/root/compiler maxxing/compiler-dev
```

Then run `autotest` inside the container (see [Testing](#7-testing-with-autotest-docker)).

### 3.2 Local build & run

The project uses Rust with dependencies:

- `lalrpop-util = 0.20.2`
- `koopa = 0.0.7`
- build dependency: `lalrpop = 0.20.2`

Locally, you can run:

```bash
cargo run -- -koopa hello.c -o hello.koopa
cargo run -- -riscv hello.c -o hello.asm
```

---

## 4. Command-Line Interface

The implementation expects arguments shaped like the following:

```bash
compiler MODE INPUT -o OUTPUT
```

Examples (as in the project README):

```bash
cargo run -- -koopa hello.c -o hello.koopa
cargo run -- -riscv hello.c -o hello.asm
```

Where:

- `MODE` is one of: `-koopa`, `-riscv`, `-perf`
- `INPUT` is a SysY source file (e.g. `hello.c`)
- `OUTPUT` is the destination file for Koopa IR text or RISC-V assembly

---

## 5. Compilation Pipeline

The codebase follows the standard compiler structure:

1. **Lexing + Parsing (frontend)** using `lalrpop`
2. **AST construction**
3. **IR generation (Koopa IR)** from AST in `src/ir_builder/`
4. **Assembly generation (RISC-V)** from Koopa IR in `src/assembly_builder/`

### 5.1 Frontend (parsing)

- `src/sysy.lalrpop` defines grammar and produces typed AST nodes in `src/ast_def/`.
- The parser skips whitespace and both line/block comments (see the `match { ... }` section).

### 5.2 IR generation (`src/ir_builder`)

Main entry:

- `ir_builder::generate_ir(comp_unit: &CompUnit) -> koopa::ir::Program`

Key mechanisms:

- maintains:
  - current function (`curr_func`)
  - current basic block (`curr_block`)
  - a stack of symbol tables for scoped variables/constants
  - break/continue target blocks for loops
- declares SysY runtime library functions at the beginning of `CompUnit::build`:
  - `getint`, `getch`, `getarray`
  - `putint`, `putch`, `putarray`
  - `starttime`, `stoptime`

### 5.3 Backend (`src/assembly_builder`)

Main entry:

- `generate_assembly(program: &Program, output_file: &mut File)`

The backend:

- dumps global allocations under `.data`
- dumps functions under `.text`
- includes:
  - lowering for Koopa IR instructions such as binary ops, load/store, calls, branches, jumps, getptr/getelemptr
  - a register allocation strategy implemented as an LRU-like cache in `MyBBValueTable`

---

## 6. Lab Progression (Lv.1 ~ Lv.9)

This section summarizes what each lab level adds/changes in the implementation, matching the work recorded in the project README and consistent with the repository’s code structure.

### Lv.1 — `main` / parser + basic IR generation framework

Deliverables:

- configure dependencies and build script (`Cargo.toml` + `build.rs` for `lalrpop` generation)
- implement the initial parser and AST wiring:
  - grammar in `src/sysy.lalrpop`
  - AST definitions in `src/ast_def/`
- create a minimal `ir_builder` module:
  - `generate_ir` produces an in-memory Koopa IR `Program`

Testing approach:

- Koopa mode first (`-koopa`) to validate parsing + IR generation

### Lv.2 — Initial code generation (Koopa IR layout traversal)

Key change:

- implement a backend named `assembly_builder` to convert Koopa IR components into assembly.
- update `main.rs` to support `-riscv` (assembly output).

Conceptual note:

- Koopa IR stores instructions in a graph-like structure; instruction IDs are used instead of a simple linear list.

### Lv.3 — Expressions

Frontend:

- extend grammar and AST cases for more expression forms
- implement recursive expression lowering in `src/ir_builder/build_expressions/`

Backend:

- implement expression instruction selection to RISC-V:
  - arithmetic ops
  - comparisons / equality:
    - lowering using `xor` + `seqz` (or `snez`)
    - `<=` and `>=` derived from negated `>`/`<` (using `seqz`)
- boolean logic:
  - use integer conversions (`!= 0`) because Koopa/this subset may not have direct logical ops

### Lv.4 — Constants and Variables

Frontend:

- support:
  - local and global declarations
  - constant expressions (constant folding)
- maintain a symbol table:
  - map identifiers to type + Koopa IR `Value`

IR generation strategy:

- compute constant expressions at compile time when both sides are known constants.

Backend:

- implement stack allocation for locals + temporaries.
- for every Koopa `load`/`store`, generate corresponding RISC-V `lw`/`sw`.
- handle naming/semantics issues discovered during testing (e.g. early returns).

### Lv.5 — Statement blocks and scope

Frontend:

- support nested blocks and scope rules:
  - manage a stack of symbol tables
  - lookup searches from inner to outer scope
- refactor IR build traits into:
  - a statement-building component
  - an expression-building component
- ensure early `return`:
  - ignore subsequent statements after a return is built

Koopa naming constraint workaround:

- avoid problematic characters in Koopa names (e.g. the README notes issues with hyphen in temporary naming),
- rely on Koopa’s renaming behavior and/or implement your own naming conventions.

### Lv.6 — `if` and short-circuit evaluation

Grammar:

- resolve the classic dangling-else ambiguity by using an unambiguous grammar structure.

IR:

- implement `if/else` using Koopa basic blocks:
  - create blocks for:
    - condition true path
    - condition false path
    - join/end
  - use Koopa `branch` and `jump` to link blocks.

Short-circuit:

- implement `exp1 || exp2` and `exp1 && exp2`:
  - for non-constant `exp1`, generate control flow to avoid evaluating `exp2` when short-circuit conditions are met
  - for constant `exp1`, fold the result directly.

### Lv.7 — `while`, break, continue

Grammar:

- extend statement syntax for:
  - `while (Exp) Stmt`
  - `break;`, `continue;`

IR:

- implement loop control with 3 blocks:
  - condition
  - loop body
  - loop exit

Semantic correctness:

- maintain two stacks:
  - break target blocks
  - continue target blocks
- when building the loop body, wire `break` / `continue` to the correct blocks.

### Lv.8 — Functions and global variables

Frontend:

- extend IR generation to handle:
  - function definitions
  - global declarations
  - function calls
  - parameter passing

Implementation strategy (from README):

- maintain a global function table
- allocate function parameters inside the function:
  - use Koopa `alloc` to create storage
  - use `store` to copy actual arguments into storage
- add runtime library decls in `CompUnit::build`

Backend:

- support:
  - global variables:
    - treat `load` source as global vs local
  - function calls:
    - prepare call arguments into ABI registers
    - save caller-saved temporaries if needed
    - restore and handle return value in `a0` (mapped to `REG_A0`)
  - stack frame layout:
    - save return address (`ra`) in prologue
    - allocate space for locals/temps and extra call args

### Lv.9 — Arrays

Frontend:

- update grammar for:
  - array declarations
  - array indexing lvalues
  - array initialization aggregates
- extend semantic checks:
  - array dimension expressions must be const for declaration
  - initialization must match alignment/layout rules
  - break/continue are validated to be inside loops

Lvalue lowering detail:

- lvalues can represent:
  - direct address (pointer)
  - value (loadable variable)
  - compile-time constant (in const context)

Backend:

- support Koopa pointer and address computations:
  - handle instructions like `getptr` and `getelemptr`
  - compute element address:
    - `base + index * sizeof(i32)`
  - load/store from computed addresses.

---

## 7. Testing with `autotest` (Docker)

The project README includes container-based testing usage:

```bash
./autotest [-koopa|-riscv|-perf] [-t TEST_CASE_DIR] [-w WORKING_DIR] REPO_DIR
```

Parameters:

- `-koopa`: test Koopa IR generation
- `-riscv`: test RISC-V assembly generation
- `-perf`: performance-related tests (treated as assembly generation in this implementation)
- `-t TEST_CASE_DIR`: directory containing `.c` / `.sy` files and matching `.out` expected outputs

Example (functional tests):

```bash
autotest -riscv -t Compiler-Lab-Test-Suites -s functional_test /root/compiler
```

If you maintain a working directory (`-w`), the test script may keep build artifacts for incremental compilation.

---

## 8. Project Code Map

Key files and modules used in this implementation:

- `src/main.rs`
  - parse command-line arguments
  - run `lalrpop` parser
  - mode dispatch to Koopa dump or RISC-V assembly generation

- `src/sysy.lalrpop`
  - SysY grammar (subset)
  - comment/whitespace skipping

- `src/ast_def/`
  - AST definitions for declarations, statements, expressions, symbols

- `src/ir_builder/`
  - `generate_ir` creates `koopa::ir::Program`
  - handles:
    - symbol table stack
    - basic block creation
    - break/continue targets
    - IR expression lowering (including short-circuit)

- `src/assembly_builder/`
  - `generate_assembly` converts Koopa IR to assembly
  - `build_assembly.rs` contains:
    - instruction lowering
    - LRU-like register cache `MyBBValueTable`
    - prologue/epilogue and calling convention handling

---

## 9. Known Limitations

The repository’s original README states:

- **Lv.8 and Lv.9** have a small number of sample cases that did not pass during the author’s local testing.
- This manual describes the implemented design and code structure; you should still run `autotest` to confirm full correctness for your environment.

---

## Appendix: Example Program Files

The project includes example SysY files such as:

- `hello.c`
- `hello0.c`
- `hello.asm` (generated)
- `hello.koopa` (generated)

---

End of document.

