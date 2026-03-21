# CPU Design Labs — English Experiment Documentation

This document summarizes all CPU-related experiments found under the `CPU/` directory of this repository. It focuses on what each experiment is supposed to achieve and what functionality is expected, without describing implementation procedures or copying any existing lab write-ups.

Experiments covered (from `CPU/README.md`):
- `exp4` — `CPU核集成设计` (integrated CPU core design)
- `exp7` — `单周期CPU（含指令扩展）` (single-cycle CPU with instruction extension)
- `exp12` — `多周期CPU（含指令扩展）` (multi-cycle CPU)

---

## Common System Operation (Switch/LED/7-Segment)

The repository-wide operation notes in `CPU/README.md` describe what should be observable on the board when running the CPU designs. Use these as the first “smoke test” for each experiment.

1. All switches are set to 0:
   - The board/Arduino 7-segment display shows a “marching” (running-light) pattern.
   - All LEDs are off.
2. `SW3` and `SW4` set to 1:
   - The 7-segment display shows a changing square pattern.
   - The LEDs corresponding to `SW3` and `SW4` light up.
3. Only `SW0` and `SW4` set to 1:
   - The board 7-segment display shows counting data.
   - The Arduino displays the lower 4 digits of the board’s 7-segment output.
4. Set `SW1` to 1 (so `SW[0,1,4] = 1`):
   - The Arduino shows the upper 4 digits of the board’s 7-segment output.
5. Set `SW2` to 1:
   - The system enters single-step debug mode.
6. Set `SW` to all 0 except `SW0=1`, then adjust `SW[7:5]`:
   - `001`: lower 2 digits keep changing
   - `010`: all 8 digits change quickly and randomly
   - `111`: lower 3 digits keep changing

---

## Experiment exp4 — Integrated CPU Core Design (`CPU核集成设计`)

### What you need to do
Integrate and connect a CPU core design so that it can run using the provided memory initialization files and drive the display/LED behavior described in the common system operation section.

### Required functionality / deliverables
1. Memory initialization
   - Load the `rom` with `SCPU_DEMO9_sign.coe`.
2. Module usage (advanced requirement)
   - The LED, 7-segment, and multiplexor modules inside the experiment must use the code you designed in previous parts, not the prebuilt `ngc` versions.

### Expected observable behavior
- With the system configured according to the common switch rules, you should observe:
  - correct LED on/off mapping,
  - correct 7-segment patterns (marching / square / counting / digit split),
  - and the single-step debug behavior when `SW2=1`.

---

## Experiment exp7 — Single-Cycle CPU with Instruction Extension (`单周期CPU（含指令扩展）`)

### What you need to do
Create a single-cycle CPU that executes the supported instruction set and connects the instruction/data path with memory and I/O so the system can be observed through LEDs and the 7-segment display as described in the common section.

### Required functionality / deliverables
1. Instruction execution control (instruction coverage)
   The `SCtrl_M.v` control logic defines the CPU’s supported operations at the control level. The single-cycle CPU is expected to handle:
   - R-type ALU operations:
     - `add`, `sub`, `and`, `or`, `xor`, `nor`, `slt`, `srl`
   - R-type control-flow:
     - `jr`, `jalr`
   - I-type arithmetic/logic:
     - `addi`, `andi`, `ori`, `xori`, `slti`, `lui`
   - Memory access:
     - `lw`, `sw`
   - Branch and jump:
     - `beq`, `bne`, `j`, `jal`
2. ROM initialization (load correct COE)
   - Use one of the following ROM/input options depending on how your ALU/ROM are set up:
     - `SCPU_DEMO9_sign.coe` (signed) or
     - `I9_mem` (unsigned)
3. Module selection (advanced requirement)
   - Replace the ROM with the “Demo31” variant:
     - `SOC_SCPU_DEMO31_sign.coe` (signed) or
     - `SOC_SCPU_DEMO31.coe` (unsigned).

### Expected observable behavior
- The same observable switch/LED/7-segment outcomes as described in `CPU/README.md` should be present.
- In particular, when the program running from ROM is active, the board outputs should remain consistent with the intended demo (e.g., digit patterns/counters and debug stepping when enabled).

---

## Experiment exp12 — Multi-Cycle CPU with Instruction Extension (`多周期CPU（含指令扩展）`)

### What you need to do
Create a multi-cycle CPU that executes the supported instruction set through a multi-state control process (FSM-based control) and correctly drives the board I/O.

### Required functionality / deliverables
1. Multi-cycle control / instruction execution coverage
   The `MCtrl.v` module defines a multi-state control flow (FSM) and the operations that it recognizes based on `Opcode` and (for some cases) function bits. The multi-cycle CPU is expected to support:
   - R-type ALU operations:
     - `add`, `sub`, `and`, `or`, `nor`, `slt`, `xor`, `srl` (selected by the function field)
   - R-type control-flow:
     - `jr`
   - I-type arithmetic/logic:
     - `addi`, `slti`, `andi`, `ori`, `xori`, `lui`
   - Memory access:
     - `lw`, `sw`
   - Branch and jump:
     - `beq`, `bne`, `j`, `jal`
2. RAM initialization (load correct COE)
   - Load `ram` with `MCPU_DEMO9_sign.coe`.
3. Module selection (advanced requirement)
   - Replace the RAM with the Demo31 variant:
     - `MCPU_DEMO31_sign.coe`.

### Expected observable behavior
- Use the common switch rules to verify output:
  - LED mapping,
  - 7-segment patterns (marching/square/counting),
  - Arduino digit split,
  - and single-step debug mode when `SW2=1`.

---

## Notes on Verification

Because this repository contains mostly hardware design files (including schematics and FPGA-oriented modules), “success” is typically verified by:
- running the configured design with the specified ROM/RAM initialization files, and
- confirming the expected LED/7-segment patterns and debug stepping behavior described in `CPU/README.md`.

If your board behavior diverges from the expected patterns, first verify the correct COE file is loaded for the corresponding experiment (`exp4`/`exp7`/`exp12`) and the switch configuration matches the common operation rules.

