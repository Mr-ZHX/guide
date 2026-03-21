# Nand2Tetris Course Projects — English Experiment Overview

This document describes the experiments under `projects/projects/`, based on the **Nand2Tetris** (“The Elements of Computing Systems”) project layout. It is written for someone who has **not** done the course: for each project it states **what you are expected to do**, **what functionality you must deliver**, and **how you know it worked**, without implementation steps or copying anyone’s lab report.

The numbered folders `01` … `13` are sequential: later projects depend on earlier ones. Folder `00` is empty in this repository snapshot (no coursework files).

---

## Project 01 — Boolean Logic (`01/`)

### What you do
Implement a library of **combinational logic chips** in the Hardware Description Language (HDL) used by the course simulator.

### What functionality you implement
Basic gates and building blocks such as (as present in this tree): multi-bit `And`/`Or`/`Not`, multiplexers and demultiplexers (`Mux`, `DMux`, and multi-way / 16-bit variants). Each chip must match the official test vectors (`.tst` / `.cmp` pairs).

### Success looks like
All supplied tests for every `.hdl` file pass in the hardware simulator.

---

## Project 02 — Boolean Arithmetic (`02/`)

### What you do
Build **arithmetic hardware** from the logic primitives you already implemented.

### What functionality you implement
1. Adders: `HalfAdder`, `FullAdder`, `Add16`, `Inc16`.
2. An **ALU** that performs the course-specified arithmetic and logic operations on two 16-bit inputs and produces status outputs as defined by the tests.

### Success looks like
All arithmetic/ALU tests pass, including variants that check ALU status behavior.

---

## Project 03 — Sequential Logic (`03/`)

### What you do
Implement **stateful (clocked) memory elements** and small RAM hierarchies.

### What functionality you implement
1. In `03/a/`: sequential primitives and small memories — e.g. `Bit`, `Register`, `PC`, `RAM8`, `RAM64`.
2. In `03/b/`: larger RAM banks built hierarchically — `RAM512`, `RAM4K`, `RAM16K`.

### Success looks like
All sequential-memory tests pass for every required module.

---

## Project 04 — Machine Language (`04/`)

### What you do
Write programs in the **Hack assembly language** for the Hack platform.

### What functionality you implement
1. **`mult/`**: `Mult.asm` — compute `R0 * R1` and store the product in `R2` under the constraints stated in the starter file (non-negative inputs, product fits in the platform’s range).
2. **`fill/`**: `Fill.asm` — an interactive program that reacts to keyboard input and updates the screen memory map (the exact behavior is defined by the official tests / platform contract).

### Success looks like
The official CPU emulator / test scripts for `Mult` and `Fill` report success.

---

## Project 05 — Computer Architecture (`05/`)

### What you do
Assemble the **CPU + memory + computer** that executes Hack machine code and connects to the screen and keyboard memory maps.

### What functionality you implement
1. **`CPU.hdl`**: instruction decode and execution for the Hack ISA, including ALU use, jumps/branches, and memory access patterns required by tests.
2. **`Memory.hdl`**: address decoding so instruction memory, RAM, screen, and keyboard appear at the correct address ranges.
3. **`Computer.hdl`**: ties CPU, ROM, and Memory into a runnable system.

### Success looks like
Integrated tests such as `ComputerAdd`, `ComputerMax`, and `ComputerRect` (and their “external” variants) pass — meaning your machine correctly runs provided `.hack` programs end-to-end.

---

## Project 06 — Assembler (`06/`)

### What you do
Write a **translator** from Hack assembly (`.asm`) to Hack machine code (`.hack`).

### What functionality you implement
Your assembler must correctly handle:
1. **A-instructions** and **C-instructions**, including all legal mnemonics.
2. **Symbols**: predefined symbols, labels, and variables.
3. The provided reference programs in subfolders (`add/`, `max/`, `rect/`, `pong/`) including **L** (“less symbolic”) variants that stress symbol resolution.

### Success looks like
For each reference `.asm`, your tool emits a `.hack` file that matches the official reference output (bit-for-bit as required by the course grader).

---

## Project 07 — Virtual Machine Translator I (`07/`)

### What you do
Implement the first half of a **VM-to-Hack translator**: translate VM commands into Hack assembly that manipulates the VM’s stack and named memory segments.

### What functionality you implement
1. **`StackArithmetic/`** programs (`SimpleAdd`, `StackTest`): stack push/pop and arithmetic/logical operations on stack values.
2. **`MemoryAccess/`** programs (`BasicTest`, `PointerTest`, `StaticTest`): `push`/`pop` for `local`, `argument`, `this`, `that`, `pointer`, `temp`, and `static` segments.

### Success looks like
Each folder’s `.tst` compares your generated assembly against the reference VM emulator behavior and reports a match.

---

## Project 08 — Virtual Machine Translator II (`08/`)

### What you do
Extend the VM translator to support **control flow** and **function call/return**, including multi-file programs.

### What functionality you implement
1. **`ProgramFlow/`**: branching and looping (`label`, `goto`, `if-goto`) — e.g. `BasicLoop`, `FibonacciSeries`.
2. **`FunctionCalls/`**: `function`, `call`, `return`, and correct handling of **frames**, **return addresses**, and (in `StaticsTest`) **per-class static variables** across `Class1.vm`, `Class2.vm`, and `Sys.vm`.

### Success looks like
All VM test suites pass, including nested calls and static-segment semantics.

---

## Project 09 — High-Level Language (Jack) — Getting Started (`09/`)

### What you do
Write small **Jack** programs and classes using the Jack language rules.

### What functionality you implement (examples present here)
- `HelloWorld`: use the standard library to print text.
- `Average`, `ConvertToBin`, `Seven`, `ComplexArrays`: small algorithmic / I/O demos.
- `Fraction`, `List`, `Square`: object-oriented examples (classes, methods, object state).
- `BitmapEditor`: a simple tool (HTML/IML scaffolding in tree) supporting bitmap editing as defined by the course materials.

### Success looks like
Programs compile with the official Jack compiler toolchain and behave as specified in the assignment text for each mini-project.

---

## Project 10 — Jack Compiler I — Syntax Analysis (`10/`)

### What you do
Build the **front end** of a Jack compiler: tokenization and parsing.

### What functionality you implement
Your analyzer reads Jack source and emits a structured representation (in this course, typically **XML**) that reflects the language grammar. The tree includes programs, classes, class-level declarations, subroutines, statements, expressions, and terminals.

### Test bundles in this tree
- `ExpressionLessSquare` — simplified grammar case.
- `Square` — full grammar case.
- `ArrayTest` — arrays in parse trees.

### Success looks like
Your output matches the official **compare files** (`MainT.xml`, `SquareT.xml`, etc.) for the supplied test programs.

---

## Project 11 — Jack Compiler II — Code Generation (`11/`)

### What you do
Complete the compiler by generating **VM code** from the parsed Jack program.

### What functionality you implement
Correct VM translation for:
- classes, fields and statics, constructors, methods, functions
- statements (`let`, `if`, `while`, `do`, `return`)
- expressions, including method calls, `this`, arrays, and operator precedence

### Sample applications in this tree
`Average`, `ConvertToBin`, `Seven`, `ComplexArrays`, `Square`, `Pong` — compiling them end-to-end is the integration test.

### Success looks like
Generated `.vm` files run correctly on the VM emulator / CPU platform (games animate, demos print expected results).

---

## Project 12 — Jack Operating System (`12/`)

### What you do
Implement the **Jack standard library** modules in Jack itself.

### What functionality you implement
The OS services used by all Jack programs, including (files present here):
- `Math.jack` — common math helpers.
- `String.jack` — variable-length character strings.
- `Array.jack` — array allocator/adapter.
- `Output.jack` — text output.
- `Screen.jack` — pixel drawing.
- `Keyboard.jack` — keyboard input.
- `Memory.jack` — heap/memory allocation (`alloc`/`deAlloc`) and memory model.
- `Sys.jack` — system services (e.g., halt/wait/error helpers as defined by the course).

Each module has a matching `*Test` folder with tests.

### Success looks like
All OS unit tests pass. The `MemoryTest` folder includes notes about diagnosing incorrect pointer aliasing — a correct `Memory.jack` must return **distinct** allocated blocks when required.

---

## Project 13 — Beyond the Course (`13/`)

### What you do
Optional **capstone** work: extend the computer with your own software or hardware-side explorations (as suggested by the course’s final chapter).

### What functionality you implement
Whatever project you choose; there is no fixed grader in this repository snapshot beyond the encouragement text in `more fun to go.txt`.

### Success looks like
You define your own acceptance criteria (a game, a tool, an optimization, a new language feature, etc.).

---

## Quick Map: Deliverable Type by Project

| Folder | Primary deliverable |
|--------|---------------------|
| `01` | Combinational HDL chips |
| `02` | Adders + ALU (HDL) |
| `03` | Registers + RAM hierarchy (HDL) |
| `04` | Hack assembly programs |
| `05` | CPU / Memory / Computer (HDL) |
| `06` | Assembler program (high-level language) |
| `07` | VM translator (Part 1) |
| `08` | VM translator (Part 2) |
| `09` | Jack applications |
| `10` | Jack syntax analyzer |
| `11` | Jack compiler (VM code gen) |
| `12` | Jack OS standard library |
| `13` | Open-ended extension |

---

*End of document.*
