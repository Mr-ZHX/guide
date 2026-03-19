# Zhejiang University — Computer Architecture Lab 2024 (Fall–Winter)

This document consolidates the six lab assignments for the 2024–2025 Fall–Winter *Computer Architecture* course (Professor Chang Rui; TAs: Zhong Zihang, Qin Jiajun). The course repository is hosted on [Gitee](https://gitee.com/crix1021/zju-computer-architecture-course-2024/) and [ZJU Git](https://git.zju.edu.cn/zju-arch/arch-fa24); lab pages are published at [ZJU Git Pages](https://zju-arch.pages.zjusct.io/arch-fa24). Some content is adapted from the *Computer System* series; see [ZJU Sys](https://git.zju.edu.cn/zju-sys) for references.

---

## Report Requirements (All Labs)

Reports must **include only** the following:

- **Design**: Design rationale with key code excerpts (do not paste entire files).
- **Assigned questions**: Answer all “思考题” (thinking questions) for that lab.
- **Reflection** (optional): Brief感想 (reflections).

Do not include unnecessary material (e.g. lengthy theory, redundant simulation/board results) unless specified. Specific labs may add extra requirements; check each lab description.

---

## Lab Overview and Deadlines

| Lab | Topic | Deadline |
|-----|--------|----------|
| **Lab 1** | Pipelined RISC-V CPU (RV32I + Forwarding) | 2024-10-10 23:59:59 |
| **Lab 2** | Pipeline Exceptions and Interrupts | 2024-10-31 18:30:00 |
| **Lab 3** | Dynamic Branch Prediction (BHT + BTB) | 2024-11-28 18:30:00 |
| **Lab 4** | L1 Cache Design | 2024-12-05 18:30:00 |
| **Lab 5** | Scoreboard Out-of-Order Pipeline | 2024-12-19 18:30:00 |
| **Lab 6** | Tomasulo Out-of-Order Execution (Optional / Bonus) | 2024-12-29 23:58:00 |

**Environment (all labs):** Verilog/SystemVerilog; Vivado 2020.2 or later; NEXYS A7 (XC7A100TCSG324). Reinstall Vivado if you removed the version used in the Computer Organization course.

---

# Lab 1 — Pipelined RISC-V CPU Design

**Deadline:** October 10, 2024, 23:59:59

## Objectives

- Review pipelined CPU design.
- Implement the RV32I instruction set.
- Understand and implement **forwarding** (bypassing) to reduce stalls.

## Instruction Set

Implement the **first 37 instructions** of the RV32I unprivileged ISA (excluding `fence`, `ecall`, `ebreak`). Reference: [RISC-V Unprivileged Spec](https://github.com/riscv/riscv-isa-manual/releases/download/Ratified-IMAFDQC/riscv-spec-20191213.pdf).

- **Integer (21):** ADDI, SLTI, SLTIU, XORI, ORI, ANDI, SLLI, SRLI, SRAI, ADD, SUB, SLL, SLT, SLTU, XOR, SRL, SRA, OR, AND, LUI, AUIPC.
- **Branch/Jump (8):** JAL, JALR, BEQ, BNE, BLT, BLTU, BGE, BGEU.
- **Memory (8):** LB, LH, LW, LBU, LHU, SB, SH, SW.

Semantics (e.g. sign-extension of immediates, PC+4 for JAL/JALR, branch target = PC + sign_ext(offset)<<1) follow the manual.

## Forwarding

**Data hazards** arise when a later instruction reads a register that an earlier instruction has not yet written back. Two approaches:

1. **Stall:** Hold PC and IF/ID, flush ID/EX (insert NOPs) until the result is available.
2. **Forwarding:** Feed the result from a later pipeline stage (e.g. EX or MEM) to the ALU or register-read stage so the dependent instruction gets the correct value without stalling.

The framework uses a **Hazard Detection Unit** (PC and IF/ID write enable, NOP insertion) and a **Forwarding Unit** (multiplexers before ALU and/or register output). You may choose which stage receives forwarded data (e.g. EX or ID) according to your pipeline timing.

**Load-use hazard:** When a load is followed by an instruction that uses the loaded value, one extra stall cycle is typically needed so the load result can be forwarded from WB to EX (or two stalls if forwarding to ID). Adjacent load and store may need one stall or extra forwarding logic if not handled by forwarding.

## Requirements

1. Implement all RV32I instructions listed above (no fence/ecall/ebreak).
2. Implement forwarding and hazard detection in the pipeline.
3. Pass simulation and board verification.

## Implementation

- Complete: `src/lab1/common/cmp_32.v`, `src/lab1/core/HazardDetectionUnit.v`, `src/lab1/core/CtrlUnit.v`, `src/lab1/core/RV32core.v`.
- HazardDetectionUnit: ports are given; implement the logic.
- CtrlUnit, RV32core, cmp_32: fill in the parts marked `to fill sth. in`.

You may adapt or extend the framework; document any changes in the report.

## Test and Validation

- Test program: `src/lab1/all_test.s` (comments give PC and register values). Verification is by checking register and PC values in simulation and on board.

## Thinking Questions

1. After adding forwarding, did you observe fewer stalls? Point to one place in the test where forwarding actually helps and show a simulation screenshot.
2. In the given framework, the comparator `cmp_32` is in the ID stage. Compare putting the comparator in ID vs EX (e.g. from a delay/timing perspective).

---

# Lab 2 — Pipeline Exceptions and Interrupts

**Deadline:** October 31, 2024, 18:30:00

## Objectives

- Learn basic RISC-V exception and interrupt handling.
- Add precise exception and interrupt support to the pipeline.

## Privilege and CSR

RISC-V has three privilege levels: **M (Machine)**, **S (Supervisor)**, **U (User)**. This lab implements **M mode** only. Implement these CSRs: `mstatus`, `mtvec`, `mepc`, `mcause`, `mtval`. Definitions and encodings: [RISC-V Privileged Spec](https://github.com/riscv/riscv-isa-manual/releases/download/Ratified-IMFDQC-and-Priv-v1.11/riscv-privileged-20190608.pdf).

## Exceptions and Interrupts

- **Exception:** event associated with an instruction (e.g. illegal instruction, access fault, ecall).
- **Interrupt:** external, asynchronous event.

On trap (exception or interrupt), hardware must:

- Save PC to `mepc` (for exception: faulting instruction; for interrupt: resume address).
- Set PC to `mtvec`.
- Set `mcause` (and optionally `mtval` per spec).
- Clear `mstatus.MIE`, save old MIE in `mstatus.MPIE`.
- Save previous privilege in `mstatus.MPP`, set privilege to M.

On **mret**: restore PC from `mepc`, restore MIE/MPIE/MPP.

## Exceptions to Implement

1. **Access fault:** Invalid memory access (e.g. write to ROM).
2. **Environment call:** `ecall`.
3. **Illegal instruction:** Invalid opcode in decode.
4. **External interrupt:** As specified in the lab material.

**Precise exception:** When an exception is taken, the pipeline must appear as if the faulting instruction caused the trap and prior instructions are committed; later instructions are squashed. Implement precise exception handling.

## Instructions to Implement

CSR and trap instructions: `csrrw`, `csrrs`, `csrrc`, `csrrwi`, `csrrsi`, `csrrci`, `ecall`, `mret`. Pseudo-instructions such as `csrr`/`csrw`/`csrs`/`csrc` are defined in the unprivileged spec and need not be implemented as separate opcodes.

## Requirements

1. Implement the CSR and trap instructions above.
2. Implement the five CSRs: mstatus, mtvec, mepc, mcause, mtval.
3. Implement the three exceptions and external interrupt with **precise exception**.
4. Pass simulation and board verification.

## Implementation

- Main work in **ExceptionUnit.v**. You may need new signals to record where the exception occurred (for `mtval`).
- **CSRRegs.v** already provides basic CSR read/write; you may need to extend it for trap handling.

**Notes:**

- Only M mode is required; no cross-privilege checks.
- You may define how current privilege is stored (only M is used).
- For `mtval`: access fault → fault address; illegal instruction → faulting instruction; others as you choose.
- `lab2_ref` contains reference test and simulation information.

## Thinking Questions

1. What is the difference between precise and imprecise exception?
2. In the test code, which instruction causes the first trap? What does the trap handler do? If you ran the test from U mode, what new exception would appear?
3. Why is the exception taken only after the instruction reaches the last stage (WB)? Could it be taken as soon as it is detected in an earlier stage? If yes, how would the exception unit need to work? If no, explain.

---

# Lab 3 — Dynamic Branch Prediction

**Deadline:** November 28, 2024, 18:30:00

## Objectives

- Understand branch prediction concepts.
- Implement **BHT (Branch History Table)** and **BTB (Branch Target Buffer)** for dynamic branch prediction.

## BHT

- Small buffer storing branch PC (or hashed PC) and **history** (taken/not taken).
- Use **2-bit** counters per entry (e.g. strong/weak taken, strong/weak not taken) to improve accuracy. Update the state on each branch resolution.
- On misprediction, flush and refetch, and update the BHT.

## BTB

- Stores **target PC** for branches that were taken. When BHT predicts “taken,” use current branch PC to index the BTB and get the target PC so that fetch can continue without a bubble.
- Update BTB only when a branch is actually taken (record branch PC → target PC).

## Pipeline Integration

- Prediction is done in **ID**; IF and ID PCs are inputs to the predictor; it outputs the predicted next PC.
- You need to modify IF fetch logic and IF/ID flush signals. Add your BHT+BTB module and connect it to **RV32core**.

## Requirements

1. Implement BHT + BTB dynamic branch prediction in the 5-stage pipeline (with forwarding).
2. Pass simulation and board verification.
3. Complete the thinking questions in the report.

## Test Program

- The test is a **sorting** program; simulation takes a long time. Run long enough (e.g. 50 μs); when `main` returns the program loops. “error” in Tcl output means the sort failed (e.g. `check` in `sort.c` fails).
- **Verification:** Simulation is used to check that branch prediction is active (e.g. fewer cycles in the loop compared to no prediction); board is used to check that the **sorted array** output is correct. With `SW[13]` and `SW[8]` on, other switches 0, the board prints the array; no single-step needed.

Reference: `lab3_ref/src/sort.c`, `lab3_ref/obj` (disassembly). For debugging, you can trace `wb_pc` to a file and compare traces with and without branch prediction to find mismatches.

## Thinking Questions

1. Show four simulation waveforms: (a) predicted taken, actual not taken; (b) predicted not taken, actual taken; (c) predicted taken, actual taken; (d) predicted not taken, actual not taken.
2. Compare dynamic vs static branch prediction: advantages and disadvantages.

---

# Lab 4 — L1 Cache Design

**Deadline:** December 5, 2024, 18:30:00

## Objectives

- Understand the role of cache in the CPU and the memory hierarchy.
- Understand the **CMU (Cache Management Unit)** and its interaction with cache and memory.
- Integrate an L1 cache into the CPU.

## Cache Policy

- **Organization:** 2-way set-associative.
- **Write policy:** **Write-back, write-allocate.**
- **Replacement:** **LRU** per set.
- **Size:** 1024 B total; block size 16 B → 32 sets (1024/16/2).

Cache line format (tag, index, offset, valid, dirty, etc.) is as in the lab figure.

## CMU

- CMU is the control unit for cache–CPU–memory interaction. It is typically implemented as a **state machine** with states such as:
  - **S_IDLE:** Normal cache read/write (hit or no cache use).
  - **S_PRE_BACK:** Prepare to write back a line (read from cache).
  - **S_BACK:** Write block back to memory (wait for memory ack; 4 cycles per access in the lab).
  - **S_FILL:** Fill block from memory into cache (wait for ack).
  - **S_WAIT:** Perform the pending access that caused the miss after fill/back.

Transitions follow the standard flow: on miss, if line is dirty then back, then fill; then retry the access.

## Requirements

1. Implement cache and CMU with write-back, write-allocate, and LRU.
2. Pass simulation and board verification.

## Implementation

- Complete `src/lab4/cache/cache.v` and `src/lab4/cache/cmu.v`. Fill in the parts marked `to fill sth. in` or `= ??`.

**Simulation:** Two setups: (1) CMU + Cache only (`code/cache/sim/sim_top.v`); (2) full CPU (`code/sim/core_sim.v`). Run (1) first, then (2). In Vivado, run sim_1 for cache-only, sim_2 for full CPU. To compare behavior without cache, you can temporarily bypass the CMU and use the original RAM and set `cmu_stall` to 0 (restore when re-enabling cache).

## Thinking Questions

1. Show and analyze **cache hit** and **cache miss with dirty line** (write-back): state transitions and cycle counts.
2. For 2-way set-associative LRU, how many bits per set are needed for a true LRU implementation?

---

# Lab 5 — Scoreboard Out-of-Order Pipeline

**Deadline:** December 19, 2024, 18:30:00

## Objectives

- Understand the **Scoreboard** algorithm for out-of-order execution.
- Understand multi-cycle functional units and out-of-order completion.
- Implement a pipeline that uses Scoreboard to schedule instructions.

## Scoreboard Stages

1. **IF (Instruction Fetch):** Fetch from memory.
2. **IS (Issue):** Issue to a functional unit if (i) a unit is free, and (ii) no other active instruction writes the same destination (avoids WAW). If structural or WAW hazard, stall issue (and eventually fetch when the buffer fills).
3. **RO (Read Operands):** Wait until source operands are ready (no earlier active instruction will write them). Scoreboard resolves RAW here; instructions may enter execution out of order.
4. **EX (Execute):** Functional unit runs; may take multiple cycles. When done, notify Scoreboard.
5. **WB (Write Back):** If no WAR hazard (no earlier-issued instruction that has not yet read an operand and has that operand as the same register as this instruction’s result), write result to the destination register. Otherwise stall until WAR clears.

## Requirements

1. Implement the Scoreboard algorithm and integrate it into the pipeline.
2. Pass simulation and board verification.

## Implementation

- Generate **Multiplier** and **Divider** IPs in Vivado (see lab document for options: Multiplier and Divider Generator with given settings; component names “multiplier” and “divider”).
- Complete `src/lab5/core/CtrlUnit.v` at positions marked `//fill sth. here`; use macros in `src/lab5/core/CtrlDefine.vh`.

**Verification:** Serial output includes **CLK_CNT**. Check that its value matches the reference after each instruction (see `src/lab5/ref/inst.S`). In simulation, observe `clk_counter` in RV32Core. **WB_ADDR** = register written in WB; **WB_DATA** = value written. Only one register write per cycle; handle multiple units completing in the same cycle (e.g. priority or delay).

## Thinking Questions

1. In the simulation waveform, point out one place where **out-of-order execution** occurs.
2. (Optional) Brief reflection on the course: what you learned and any suggestions (no impact on grade).

---

# Lab 6 — Tomasulo Out-of-Order Execution (Optional / Bonus)

**Deadline:** December 29, 2024, 23:58:00

**This lab is optional; bonus points count toward the usual grade (and do not overflow the total).**

## Objectives

- Understand **register renaming** and its role in removing WAR/WAW.
- Understand the **Tomasulo** algorithm.
- Implement a simple out-of-order processor following the textbook Tomasulo description.

## Register Renaming

- **RAW:** Read After Write — reader depends on a prior writer; solved by forwarding or waiting.
- **WAW:** Write After Write — two instructions write the same register; order of writes must be preserved.
- **WAR:** Write After Read — later instruction writes a register that an earlier instruction reads; the read must see the old value.

**Register renaming** maps architectural registers (AR) to physical registers (PR) so that each write targets a “new” name, eliminating WAW and WAR. A **RAT (Register Alias Table)** keeps the current AR → PR mapping.

## Tomasulo Algorithm

- **Reservation Stations (RS):** Hold issued instructions until operands are ready. Operands can be either values (Vj, Vk) or tags (Qj, Qk) pointing to the RS that will produce them. RS entries act as virtual physical registers.
- **Common Data Bus (CDB):** When a functional unit or load completes, it broadcasts the result on the CDB; the register file and all RS entries waiting on that tag update and clear the tag.
- **Load/Store:** Can use a separate load/store buffer; the lab may require in-order memory or a simple policy (e.g. all memory in order, or only one load/store RS entry) to avoid complex memory disambiguation.

**Issue:** Check for free RS. For FP: read operands from reg file or set Qj/Qk from RAT; update RAT for destination. For load/store: similar for address; for store, also record value (or its tag).  
**Execute:** When operands ready, run in FU; for load, compute address and read memory.  
**Write-back:** FP and load: when CDB free, broadcast result; update reg file and RAT; clear RS. Store: write to memory, then release RS. **Arbitrate** when multiple units complete in the same cycle (e.g. priority as in Lab 5).

## Requirements

1. Implement the textbook Tomasulo algorithm. **Minimum for bonus:** register renaming that eliminates WAR/WAW (6 pts). Extra points for Load/Store Queue, RS scheduling policy, ROB, out-of-order memory (see lab for point breakdown).
2. Simulation only; no board.

## Implementation

- Complete `RV32core.v` and `Regs.v`; implement the **RS** module according to the interface in `RV32core.v` (parameterized design `#(...)`). You may define your own RS and plug it in.
- Use **both** Lab 3 and Lab 5 test programs: `ram.mem`/`rom.mem` for Lab 5; `lab3_ram.mem`/`lab3_rom.mem` for Lab 3. Add all four to the project; switch by changing the init file names in `ROM_D.v` and `RAM_B.v`.
  - Lab 5 test: use simulation to show that register renaming is working.
  - Lab 3 test: provide a simulation screenshot showing the program reaching the infinite loop at 0x21C and the time taken.
- Submit: design description, simulation screenshots, and source code (zipped) together with the report.

---

# Summary Table

| Lab | Main topics | Deliverables |
|-----|-------------|--------------|
| 1 | RV32I pipeline, hazard detection, forwarding | CPU in cmp_32, HazardDetectionUnit, CtrlUnit, RV32core; sim + board |
| 2 | CSR, exceptions, interrupts, precise trap | ExceptionUnit (+ CSR/trap logic); sim + board |
| 3 | BHT, BTB, dynamic branch prediction | BHT+BTB module in pipeline; sim + board |
| 4 | L1 cache, CMU, write-back, write-allocate, LRU | cache.v, cmu.v; sim + board |
| 5 | Scoreboard, out-of-order issue/execute/WB | CtrlUnit (Scoreboard); sim + board; CLK_CNT check |
| 6 | Tomasulo, register renaming, RS, CDB | RV32core, Regs, RS; sim only; report + code |

---

*This document is a consolidated English version of the six lab handouts. For the latest instructions, figures, and environment details, refer to the original Chinese docs in the `docs/` folder and the course Git repositories.*
