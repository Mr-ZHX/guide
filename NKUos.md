# OS Experiments (uCore): English Experiment Descriptions (Total)

This document consolidates the five OS lab experiment description documents (Lab 1 to Lab 5) into a single file.
It only specifies experimental requirements, the functions/modules to understand or implement, and what needs to be implemented.

---

## Experiment 1: Kernel Entry and Timer Interrupt Handling (uCore OS Lab)

### Experimental Requirements
- Use the provided OS (uCore) codebase under the `OS/` directory.
- For written exercises: read the specified source files and answer the given questions in English (markdown is acceptable).
- For programming exercises:
  - Modify the OS interrupt/exception handling code so that the system prints `100 ticks` after every 100 clock interrupts.
  - After printing 10 lines of `100 ticks`, call `shut_down()` (declared in `sbi.h`) to power off.
  - Build and run the system (the lab states `make qemu`), and verify that the output pattern matches the specification (roughly one `100 ticks` line per second; 10 lines total).
- Submission: submit an updated source code package and a short report describing your implementation process and the clock interrupt handling flow.

### Functions (Modules to Understand/Touch)
- Kernel entry and boot flow:
  - `kern/init/entry.S` (specifically `kern_entry`)
  - The instruction `la sp, bootstacktop`
  - The tail call `tail kern_init`
- Timer interrupt/exception handling:
  - `kern/trap/trap.c`:
    - the `trap` handling path / timer-interrupt branch
    - where to call `print_ticks`
    - the point where to invoke `shut_down()` from `sbi.h`
- Interrupt exception entry mechanism (written analysis):
  - `trapentry.S`:
    - `SAVE_ALL` / `RESTORE_ALL`
    - `__alltraps` and related trap entry code

### What Needs to Be Implemented

#### Basic Exercise 1: Understand Kernel Entry Instructions (Written)
1. Read `kern/init/entry.S` and explain:
   - What does `la sp, bootstacktop` do?
   - What is the purpose of `tail kern_init`?
2. Connect the instructions to the kernel startup flow (why these steps are needed for correct execution).

#### Basic Exercise 2: Implement Timer Interrupt Behavior (Programming)
1. In `kern/trap/trap.c`, complete the clock/timer interrupt handling code so that:
   - After every 100 timer interrupts, the OS prints a line containing `100 ticks`.
   - After printing 10 lines total, call `shut_down()` from `sbi.h` to shut down the machine.
2. Ensure your implementation:
   - Correctly counts timer interrupts (100 interrupts per print).
   - Correctly counts printed lines (10 lines before shutdown).
   - Schedules the next timer event according to the lab framework (if required by the provided code structure).
3. Compile and run with `make qemu`, and verify the expected output cadence and termination.

#### Challenge 1: Describe and Understand the Interrupt Flow (Written)
Answer in English:
- Describe ucore interrupt/exception handling flow starting from the moment the exception/interrupt is generated.
- What is the purpose of `mov a0, sp`?
- In `SAVE_ALL`, where are saved registers placed on the stack (how to determine the exact locations)?
- For any interrupt, does `__alltraps` need to save all registers? Provide reasoning.

#### Challenge 2: Understand Context Switch Mechanism (Written)
Answer in English:
- Explain what `csrw sscratch, sp` and `csrrw s0, sscratch, x0` implement and their purposes.
- `SAVE_ALL` stores CSRs like `stval` and `scause`, but `RESTORE_ALL` does not restore them. Why is storing them still meaningful?

#### Challenge 3: Complete Exception Interrupt Handling (Programming)
Extend `trap.c` exception handling:
- Handle `CAUSE_ILLEGAL_INSTRUCTION`:
  - Output the exception type (e.g., "Illegal instruction").
  - Output the exception instruction address.
  - Update `tf->epc` so execution continues after the offending instruction (account for instruction size as required).
- Handle `CAUSE_BREAKPOINT`:
  - Output the exception type (e.g., "breakpoint").
  - Output the exception instruction address.
  - Update `tf->epc` accordingly.

---

## Experiment 2: Physical Memory Allocation Algorithms (First-Fit, Best-Fit, Buddy, SLUB)

### Experimental Requirements
- Complete `Exercise 0` by filling in the code dependency from the previous lab (this lab states it depends on Experiment 1).
- Use the code and interfaces provided by the OS (uCore) project.
- Produce a markdown/text report including:
  - What you designed/implemented for each exercise.
  - For each basic exercise, answer the provided improvement questions.
  - After finishing, analyze the reference implementation provided in `ucore_lab` and describe the differences between your implementation and the reference.
  - List important knowledge points and relate them to corresponding OS principles.
  - List OS principles knowledge points that are important but not directly covered by the lab tasks.
- For programming challenges: provide sufficiently thorough test cases and (where required) design documents.

### Functions (Modules to Inspect/Modify)
- Physical memory allocator core implementation:
  - `kern/mm/default_pmm.c`
  - Functions to study:
    - `default_init`
    - `default_init_memmap`
    - `default_alloc_pages`
    - `default_free_pages`
- Additional allocator implementations (as required by challenges):
  - Buddy system allocator logic
  - SLUB allocator logic (two-level allocation concept)

### What Needs to Be Implemented

#### Exercise 0: Fill the Dependency Code (Programming Integration)
- This experiment depends on Experiment 1.
- Insert/fill your Experiment 1 code into the current lab template at the specified markers (the lab indicates regions annotated with `LAB1`).

#### Exercise 1: Understand First-Fit Contiguous Physical Memory Allocation (Written)
- Read the tutorial and analyze relevant code in `kern/mm/default_pmm.c`.
- Describe:
  - How the first-fit contiguous physical page allocation works end-to-end.
  - The role of each mentioned function (`default_init`, `default_init_memmap`, `default_alloc_pages`, `default_free_pages`).
- Answer whether your first-fit design has further optimization space.

#### Exercise 2: Implement Best-Fit Contiguous Physical Memory Allocation (Programming)
- Implement the best-fit contiguous physical memory allocation algorithm by following the first-fit structure in `kern/mm/default_pmm.c`.
- Constraints:
  - No strict time/space complexity requirements; passing tests is the primary goal.
- Report requirements:
  - Briefly describe design/implementation.
  - Explain clearly how the algorithm allocates and frees physical memory.
  - Answer whether the best-fit algorithm has further optimization space.

#### Challenge: Implement Buddy System (Programming)
- Implement a buddy system allocation algorithm:
  - Manage available memory as blocks.
  - Each block size is a power of two: `Pow(2, n)` (1, 2, 4, 8, 16, 32, 64, 128, ...).
- Reference is provided by the lab.
- Requirements:
  - Implement buddy system in uCore.
  - Provide relatively comprehensive tests proving correctness.
  - Provide a design document.

#### Challenge: Implement SLUB-like Allocator for Arbitrary-Sized Units (Programming)
- Implement a simplified SLUB-like concept:
  - Two-level architecture:
    - Level 1: page-size-based allocation.
    - Level 2: arbitrary-size allocation built on top of Level 1.
  - Simplification is allowed as long as the main idea is shown.
- Reference is provided by the lab.
- Requirements:
  - Provide sufficiently thorough tests proving correctness.
  - Provide a design document.

#### Challenge: Hardware Usable Physical Memory Range Acquisition (Written)
- Thinking question: if the OS cannot know the current hardware usable physical memory range in advance, how can it obtain it?

---

## Experiment 3: Virtual Memory and Page Replacement (FIFO/Clock/Large Page/LRU)

### Experimental Requirements
- Follow the lab template and implement only the requested missing parts.
- Exercise 0 is a dependency: integrate required code from previous experiments before core tasks.
- For each basic exercise:
  - Written tasks: answer questions in your report.
  - Programming tasks: implement required functions so provided tests pass.
- For LRU challenge: provide detailed design/analysis and a test plan (bonus if implemented correctly).

### Functions (Modules to Inspect/Modify)
- Page replacement algorithm traces (written analysis):
  - FIFO logic, mainly organized in `kern/mm/swap_fifo.c`.
- Paging mechanism analysis (written):
  - `get_pte()` in `kern/mm/pmm.c`:
    - compare similar code patterns in the context of `sv32`, `sv39`, `sv48`.
- Page fault handling (programming):
  - `do_pgfault()` in `mm/vmm.c`
- Clock replacement algorithm (programming):
  - `mm/swap_clock.c`
- Paging mapping mode understanding (written):
  - differences between large-page mapping vs hierarchical page tables.

### What Needs to Be Implemented

#### Exercise 0: Fill Dependencies (Programming Integration)
- Depends on Experiment 1 and 2.
- Fill required code from previous labs into the current template at markers labeled `LAB1` and `LAB2`.

#### Exercise 1: Understand FIFO Page Replacement (Written)
- Under FIFO, describe the processing path of a page from being swapped in to being swapped out.
- Requirements:
  - Correctly identify at least 10 different functions/macros participating in the in->out path.
  - Each must reflect actual execution impact; avoid functions like `assert` or output-only functions such as `cprintf`.
  - Ensure coverage of the full in-to-out process, not only swap-in.

#### Exercise 2: Deep Understand Different Paging Modes (Written)
- Analyze `get_pte()` and answer:
  - Why two similar code sections appear similar (based on `sv32`, `sv39`, `sv48` context).
  - Current `get_pte()` mixes lookup and allocation: is this design good? Should split them?

#### Exercise 3: Map Unmapped Addresses to Physical Pages (Programming)
- Complete `do_pgfault()` in `mm/vmm.c`:
  - Map a physical page to an unmapped virtual address.
  - Set permissions according to the VMA permissions.
  - Map using the page table specified by the memory control structure (`mm`), not the kernel page table.
- Report requirements/questions:
  - Describe how PDE/PTE components can help implement page replacement.
  - If page fault handling itself triggers a page access exception, what hardware steps occur?
  - Explain whether each `Page` array entry corresponds to PDE/PTE entries; if yes, what is the relationship.

#### Exercise 4: Implement Clock Page Replacement (Programming)
- Implement Clock page replacement in the provided framework in `mm/swap_clock.c`.
- Report requirements:
  - Explain your design/implementation approach.
  - Compare Clock vs FIFO (behavioral differences).

#### Exercise 5: Understand Page Table Mapping Strategy (Written)
- If using a "large page" mapping strategy instead of hierarchical page tables:
  - Advantages/benefits
  - Disadvantages/risks

#### Challenge: Implement LRU Page Replacement (Programming + Report)
- Implement LRU page replacement without worrying about efficiency.
- Provide detailed design, analysis, and testing report.

---

## Experiment 4: Process Management and Context Switching (alloc_proc / do_fork / proc_run)

### Experimental Requirements
- Use the uCore OS lab template provided in `OS/lab4`.
- For exercises marked “need coding”:
  - Implement missing functions so the project builds and tests/run output match expectations.
  - After coding, provide a short English report describing your design/implementation process and answering specific questions.
- For exercises marked “written analysis” / “challenge”:
  - Provide clear written answers in English (markdown acceptable).
- Complete exercises in order:
  - Exercise 1 is the foundation.
  - Exercise 2 creates a new kernel thread via `do_fork`.
  - Exercise 3 performs switching via `proc_run`.

### Functions (Modules to Inspect/Modify)
- Process structure allocation and initialization:
  - `alloc_proc` in `kern/process/proc.c`
- Kernel thread creation logic:
  - `do_fork` and called helpers in `kern/process/proc.c`
  - Mentioned helpers:
    - `setup_kstack`
    - `copy_mm`
    - `copy_thread`
    - process list insertion/linking
    - `wakeup_proc`
- Context switching:
  - `proc_run`:
    - interrupt control macros `local_intr_save(x)` / `local_intr_restore(x)` from `/kern/sync/sync.h`
    - page table switching via `lcr3(unsigned int cr3)` from `/libs/riscv.h`
    - low-level context switch in `switch.S` with `switch_to()`

### What Needs to Be Implemented

#### Exercise 1: Allocate and Initialize `alloc_proc` (Coding)
- Implement `alloc_proc` so that it allocates a new `struct proc_struct` and initializes at least:
  - `state`, `pid`, `runs`, `kstack`, `need_resched`, `parent`, `mm`, `context`, `tf`, `cr3`, `flags`, `name`
- Report question:
  - Explain the meaning and role of `struct context context` and `struct trapframe *tf`.

#### Exercise 2: Allocate Resources for a New Kernel Thread (Coding)
- Complete the process/thread creation in `do_fork`:
  - Call `alloc_proc`
  - Allocate a kernel stack for the new kernel thread
  - Copy/prepare memory management info as required by the framework
  - Copy the original process execution context to the new process
  - Add the new process to process lists/index structures
  - Wake up the new process
  - Return new process id
- Report question:
  - Whether uCore assigns a unique id to each newly forked thread; justify.

#### Exercise 3: Implement `proc_run` Switching (Coding)
- Implement `proc_run(struct proc_struct *proc)` following the required steps:
  - Do nothing if `proc == current`
  - Disable interrupts via `local_intr_save(x)` / `local_intr_restore(x)`
  - Set `current` to the target process
  - Switch page tables with `lcr3(next->cr3)` (or equivalent)
  - Perform context switching via `switch_to()` in `switch.S`
  - Re-enable interrupts after the switch
- Report question:
  - How many kernel threads are created and run during this lab.

#### Challenge: Explain Interrupt Switch Semantics (Written)
- Explain how `local_intr_save(intr_flag); ... local_intr_restore(intr_flag);` implements interrupt enable/disable behavior in this framework.

---

## Experiment 5: User Programs and Advanced Process Memory (execv / fork / COW)

### Experimental Requirements
- Use the uCore OS lab template under `OS/lab5`.
- Resolve dependencies first:
  - Lab 5 depends on Experiments 2/3/4.
  - Fill code from previous experiments into the specified `LAB2` / `LAB3` / `LAB4` regions.
  - You may need to improve earlier code so Lab 5 tests run correctly.
- For programming tasks:
  - Implement specified missing functions so tests pass.
  - Provide a short English report describing your design/implementation process.
- For written analysis tasks:
  - Answer questions clearly in English (markdown acceptable).
- For challenges:
  - Provide source code, tests, and a design report. The COW challenge is marked as a big challenge.

### Functions (Modules to Inspect/Modify)
- Loading and executing user programs:
  - `do_execv` (calls `load_icode`)
  - `load_icode` in `kern/process/proc.c` (complete step 6)
  - User-mode `trapframe` initialization
- Process creation and memory duplication:
  - `do_fork`
  - `copy_range` in `kern/mm/pmm.c` (per lab description)
- COW memory semantics:
  - Required behavior for Copy-on-Write
- Written system call flows:
  - fork / exec / wait / exit

### What Needs to Be Implemented

#### Exercise 0: Fill in Previous Experiments (Programming Integration)
- Depends on Experiments 2/3/4.
- Fill implementations into the lab5 template at regions marked `LAB2` / `LAB3` / `LAB4`.

#### Exercise 1: Load and Execute an ELF Program (Coding)
- `do_execv` calls `load_icode` to load an ELF executable already present in memory.
- Complete step 6 of `load_icode`:
  1. Create the required user address space (code/data/stack, etc.).
  2. Initialize `proc_struct` user-mode `trapframe` fields correctly.
  3. Ensure that when scheduled, execution starts at the program entry address from the ELF header.
- Report requirements:
  - Briefly explain your design/implementation steps.
  - Describe the full chain from when uCore picks the user process in `RUNNING` state to when the first instruction executes.

#### Exercise 2: Fork and Copy User Memory (Coding)
- `do_fork` copies the valid parts of the parent's user address space into the child.
- Implement `copy_range` in `kern/mm/pmm.c` so the copy works correctly.
- Report requirements:
  - Briefly explain your design/implementation process.
  - Provide an overview design for implementing Copy-on-Write (COW).
- COW expectation:
  - Parent and child can initially share pages read-only.
  - On write, the system copies so the writing process gets its own private page.

#### Exercise 3: Analyze fork/exec/wait/exit (Written, No Coding)
- Read and analyze the source code to understand:
  - the execution flow of fork/exec/wait/exit,
  - which operations happen in user mode vs kernel mode,
  - how user/kernel interleave,
  - how kernel results return to user programs.
- Provide a process lifecycle diagram:
  - start/end states and the transitions between them
  - events/function calls that trigger each transition.

#### Challenge 1: Implement Copy-on-Write (COW) Mechanism (Big Challenge, Coding + Report)
- Implement COW in uCore:
  - Provide implementation source code.
  - Provide test cases and a design report with state transitions in the COW scenario (finite-state-machine style explanation).
- Requirements:
  - When parent creates a child, parent’s user pages are set to read-only.
  - Child can share those pages initially.
  - When either process writes to a shared page, page fault handling should trigger copying so both processes end up with separate pages.
- Additionally, reference the idea from [dirtycow.ninja](https://dirtycow.ninja/) and explain whether/how you can simulate the error and solution in uCore’s COW implementation.

#### Challenge 2: Preloading Time and Loading Differences (Written)
- Explain when the user program is preloaded into memory in this lab environment.
- Explain how it differs from typical OS program loading and why.

