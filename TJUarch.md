# Experiment Description (English): Dynamic Instruction Scheduling with Tomasulo

## Project Overview
This project is a software simulator that implements the Tomasulo algorithm (dynamic instruction scheduling / out-of-order execution). It is written in Rust and is organized so that the instruction parsing layer is separated from the CPU/Tomasulo execution core. A tracing mechanism is used to record and observe how instructions progress through the simulation.

The simulator supports a subset of instructions and includes a simple memory model for `LOAD`-type operations. It does not implement cache behavior in this version. The instruction stream is parsed by a dedicated `parser` component, and simulation progress is recorded via a `trace` mechanism. A script runner (e.g., `justfile`/`Makefile`) is provided to generate workloads and execute the simulations.

## Experimental Objectives
1. Understand and become familiar with a Tomasulo-based out-of-order execution algorithm.
2. Compare the performance of the Tomasulo out-of-order approach against a single-cycle in-order CPU baseline.

## Experimental Scope and Assumptions
The experiment models a superscalar out-of-order processor based on Tomasulo, where each cycle can fetch/issue up to `N` instructions. The implementation focuses on modeling the dynamic dispatch/scheduling mechanism.

To simplify the study of scheduling effects, the baseline experimental assumptions are:
- Perfect cache behavior
- Perfect branch prediction

## Course Experiment Requirements
The assignment requires modeling a Tomasulo-style out-of-order superscalar processor with emphasis on the *dynamic scheduling* mechanism. In particular:
- Dynamic scheduling: the simulator must reflect how instructions are issued, queued, executed, and result-broadcast when operands become available.
- Multi-issue: each cycle can fetch/dispatch/issue `N` instructions (i.e., compare different issue widths such as single-issue vs multi-issue).
- Focus scope: perfect cache and perfect branch prediction are assumed, so the experiment studies scheduling effects rather than cache/misprediction behavior.
- Baseline comparison: performance must be compared against a simple single-cycle (baseline) processor.
- Bonus (if applicable): extend the model by combining cache behavior with Tomasulo.

## Functional Requirements (What You Need to Implement)
In order to satisfy the experiment requirements, the following functional blocks should be implemented in the simulator:

### 1) Instruction Flow Control (per cycle)
- An issue/dispatch stage that can accept up to `issue_nums` instructions per cycle.
- An execution stage that starts an operation only when:
  - its operands are ready, and
  - an appropriate execution unit is available.
- A result broadcast mechanism (CDB-like) so completed execution updates dependent waiting instructions.
- A commit/write-back stage that makes completed results visible in the architectural state (e.g., register file / memory).

### 2) Dependency Handling and Hazard Avoidance
- Dependency tracking between instructions (especially handling RAW hazards by waking up consumers when producers finish).
- Register renaming support so that WAR and WAW hazards are eliminated at the dynamic scheduling level.

### 3) Microarchitectural Structures
- Reservation stations (or equivalent queues) for buffering issued instructions until operands are ready.
- A register status/tag mapping structure that tells, for each architectural register, which in-flight producer will generate its next value.
- Execution units for the modeled operation types.

### 4) Instruction Set Coverage (as required)
The assignment conceptually requires abstract instruction types; this project realizes them as concrete instructions:
- Arithmetic: `ADD`, `SUB`, `MUL`, `DIV`
- Memory: `LD`/`LOAD`-type instruction using a memory model
- Control: `JUMP` (branch-like control flow) under the assumption of perfect branch prediction

### 5) Latency Model
- Each instruction type must have a modeled execution latency in cycles, used by the simulator to advance execution.
- The performance study must use the latency model consistently across the baseline and Tomasulo variants.

### 6) Performance Evaluation
- Implement or provide a single-cycle baseline simulator.
- Run experiments with different issue widths (`1`, `2`, `4`, `8` per cycle) over multiple instruction counts (e.g., `100/500/1000/5000`).
- Record the total execution cycle counts to compare results across configurations.

### 7) (Bonus) Cache + Tomasulo Extension
- If the bonus is enabled, extend the memory subsystem to include cache behavior and integrate it with the Tomasulo scheduling flow.

## Instruction Set and Latency Model
The simulator models the following instructions:
- `ADD`, `SUB`, `MUL`, `DIV`
- `LD`/`LOAD` (load / load-word style instruction)
- `JUMP` (jump/branch-like control flow)

The execution latencies (in cycles) used for performance evaluation are:
- `ADD`: 2 cycles
- `SUB`: 2 cycles
- `MUL`: 12 cycles
- `DIV`: 24 cycles
- `LD`/`LOAD`: 2 cycles
- `JUMP`: 1 cycle

## Core Algorithm: Tomasulo Dynamic Scheduling
Tomasulo’s central ideas used in this simulator are:
1. Track instruction dependencies and readiness. As soon as operands become available, the instruction can execute to reduce the impact of RAW (Read After Write) hazards.
2. Use register renaming to eliminate WAR (Write After Read) and WAW (Write After Write) hazards.

### High-level Per-Cycle Flow
Each simulation cycle performs the following conceptual steps:
1. Write execution results to the Common Data Bus (CDB), and broadcast them to waiting reservation stations.
2. Commit completed operations (when allowed by the scheduling/state rules of the implementation).
3. Increase the cycle counter.
4. Issue up to `issue_nums` instructions for that cycle (multi-issue scheduling).
5. Execute instructions already resident in reservation stations whose operands are ready and for which execution units are available.

### Issuing and Register Renaming
During the issue stage:
- Operands are mapped to reservation stations.
- Destination registers are associated with internal tags (rename state), so that later dependent instructions can wait for the correct producing instruction rather than the architectural register name.
- When an instruction finishes, its result is broadcast on the CDB, which updates the operands of dependent reservation stations.

### Execution and Result Broadcast
During execution:
- Reservation stations whose operands are all ready can start execution if an execution unit is free.
- After execution completes, the result is placed onto the CDB so dependent instructions can wake up and proceed.

## Test Case Generation
Because the software simulator may not use the course-provided test cases directly, this project includes a generator script (`gen.rs`) that creates:
- An instruction file (e.g., `inst.txt`)
- A memory/data file (e.g., `data.txt`)

The generator randomly produces instruction sequences with configurable program lengths and fills memory with random values.

## Validation / Evaluation Plan
To meet the experimental requirements, the implementation should be validated in two dimensions:

1. Correctness verification
   - Use the generated instruction + data files (or provided test cases).
   - Ensure the simulator can parse instructions, execute them, and update architectural state (registers/memory) according to the modeled semantics.
   - Use tracing/logging output (if available) to inspect instruction timing and dependency wakeups.

2. Performance measurement
   - Compare the total cycle count of a `single_cycle` baseline versus the Tomasulo out-of-order model.
   - Evaluate multiple issue widths (per cycle) to study the impact of dynamic dispatch:
     - `1`, `2`, `4`, `8` issues per cycle
   - Evaluate multiple program lengths (instruction counts), e.g. `100/500/1000/5000`, using the generator’s configurable parameters.

## How to Run
The project provides `Makefile`/`justfile` targets to run the simulator:

```bash
make gen
make tomasulo
make single_cycle
```

To change the generated workload size, modify the parameters in `bin/gen.rs` (the instruction generator).

