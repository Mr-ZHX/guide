# NNU (Nanjing Normal University) Computer Organization Course Design — Overview (English)

Repository name: **`JiZuKeShe--NNU-2024`** (“计算机组成原理” / computer organization **course design**, **2024** cohort at **NNU**).

This tree is **one integrated CPU project** in **Verilog**, split into **folders per hardware block** (each with a `.v` source and a **Quartus `.vwf` waveform** file). There are **no separate `lab1` / `lab2` directories**; the “labs” are best understood as **milestones** corresponding to those blocks.

This document explains **what the project is for** and **what each part is supposed to accomplish**, for someone with no prior context. It does **not** teach RTL coding. It does **not** copy informal README remarks (e.g. tool versions, LLM assistance, author disclaimers) as if they were specifications.

**Tooling (from `README.md`):** **Quartus II 9.2** is mentioned for development/simulation.

---

## Relationship between README claims and the visible RTL

The **`README.md`** states that the design is a **five-stage pipelined CPU** with **forwarding**, **NOP insertion**, and **static prediction** for branches/jumps.

The **top-level `CPU_1.v` in this checkout** wires **InstructionFetch → Decode → Register file → ALU → data memory (Store)** with **no explicit inter-stage pipeline registers** between those units. Operationally it behaves like a **single-cycle (or tightly coupled) datapath**: each **PC update** corresponds to completing the path for one instruction, rather than a classic 5-stage overlap of multiple instructions.

**Takeaway:** Treat the README as the **course’s intended feature list**; treat **`CPU_1.v`** as the **actual integration style** shipped here. Your instructor’s handout may require a **true** pipelined implementation even if this reference uses a simpler timing model.

---

## Milestone A — Arithmetic logic unit (`ALU/`)

**Goal:** A **32-bit ALU** driven by a small **ALU control** code.

**Functionality:** Perform the operations required by the rest of the CPU (e.g. **add / subtract**, **unsigned/signed comparisons**, **bitwise OR**, **overflow** and **zero** flags where applicable). The exact encoding of `ALUctr` is defined by how **`Decode`** drives it.

**Outcome:** Correct combinational (or specified clocked) behavior verified by simulation.

---

## Milestone B — Register file (`Registers/`)

**Goal:** A **32 × 32-bit** GPR file in the **MIPS-style** spirit.

**Functionality:** **Two read ports** (addresses `Ra`, `Rb`) and **one write port** (`Rw`, `busW`) with **write enable**; **asynchronous read** and **edge-triggered write** (as in the provided module); optional **asynchronous reset** clearing all registers.

**Outcome:** Programs can source two operands per cycle and write back one result when `RegWr` is asserted (subject to overflow-related gating in the top level).

---

## Milestone C — Instruction fetch (`InstructionFetch/`)

**Goal:** **PC** logic plus **instruction memory**.

**Functionality:** Hold **PC**; each cycle **fetch** a 32-bit word; **decode fields** into `OP`, `Rs`, `Rt`, `Rd`, `imm16`, `func`. **Update PC** as:

- **Sequential:** `PC + 4`
- **Branch:** when branch is taken (e.g. **`beq`** with **ALU zero**), PC-relative target
- **Jump:** **`j`**-style absolute (upper PC bits + 26-bit word index)

The module includes a **hard-coded demo program** (e.g. **sum 1…10** using **`addiu`**, **`add`**, **`beq`**, **`j`**).

**Outcome:** Instruction stream is produced for simulation and board bring-up.

---

## Milestone D — Control / decode (`Decode/`)

**Goal:** **Main decoder** mapping **`OP`** (and **`func`** for R-type) to **datapath control signals**.

**Supported instruction families (as encoded in this repo):**

- **R-type:** e.g. **`add`**, **`addu`**, **`sub`**, **`subu`**, **`slt`**, **`sltu`**
- **I-type:** **`addiu`**, **`ori`**, **`lw`**, **`sw`**, **`beq`**
- **J-type:** **`j`**
- **Default:** all controls low → effectively **NOP**

Signals produced include **`Branch`**, **`Jump`**, **`RegDst`**, **`ALUsrc`**, **`ALUctr`**, **`MemtoReg`**, **`RegWr`**, **`MemWr`**, **`ExtOp`** (zero- vs sign-extend for immediates).

**Outcome:** Each opcode selects the correct register ports, ALU operation, memory access, and write-back source.

---

## Milestone E — Data memory (`Store/`)

**Goal:** Simple **RAM** for **load/store**.

**Functionality:** **Asynchronous read** by address; **synchronous write** when **`WrEn`** is high; address space truncated to **256 words** using the **low 8 bits** of the address in this design.

**Outcome:** **`lw` / `sw`** can move data between memory and registers.

---

## Milestone F — Top-level CPU (`CPU_1/`)

**Goal:** **Integrate** all blocks and expose **observable signals** for simulation or hardware debug (e.g. **ALU result**, **PC**, register bus values, **control bus**, memory read data).

**Additional behavior in this top:**

- **`Ext32`:** 16-bit immediate **zero- or sign-extension** muxed into the ALU path.
- **Register write:** **`RegWr`** may be **masked by overflow** (`RegWr_dealed`) so some overflows suppress a register write.
- **Write-back mux:** Choose **ALU result** vs **memory** for **`busW`**.

**Outcome:** End-to-end execution of the program in **`InstructionFetch`** (and any program you load there) under the timing model implied by the negedge-clocked elements.

---

## Verification artifacts

Each subdirectory’s **`.vwf`** file is a **Quartus waveform** setup for exercising that block or the full CPU. Your course will expect you to **run simulations**, capture waveforms, and possibly demonstrate on **FPGA** per local requirements.

---

## Summary table (engineering blocks)

| Folder / unit | Responsibility |
|---------------|----------------|
| `ALU/` | 32-bit ALU + flags |
| `Registers/` | Dual-read / single-write register file |
| `InstructionFetch/` | PC, IMEM, branch/jump PC update |
| `Decode/` | Opcode → control signals |
| `Store/` | Data memory |
| `CPU_1/` | Top-level integration + immediates + write-back |

---

## Using this repository responsibly

The author notes this is a **minimal course project** and may **not** match another school’s spec. **Do not** treat it as a universal solution; align **ISA subset**, **timing (single-cycle vs pipeline)**, and **hazard handling** with **your** official assignment.

---

*This overview is based on module boundaries and port lists in the Verilog sources only, not on external lab reports.*
