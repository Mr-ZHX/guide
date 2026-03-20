# riscv-pke (hust-pke / Proxy Kernel for Education): English Experiment Descriptions (Total)

This document summarizes the lab tasks implied by the `hust-pke` repository (based on README narrative and, more importantly, on in-code `@labX_Y` markers, `TODO(labX_Y)`, and implementation hints).

It focuses on:
- Experimental requirements
- Functions/modules to understand
- What needs to be implemented / completed

It intentionally avoids including any “completed report” results/answers.

---

## Project Overview (What PKE Labs Are About)

PKE builds a “just-enough” proxy kernel that upgrades its capability step-by-step so that given user applications can execute smoothly. In each lab:
1. You follow how the user application interacts with the proxy kernel.
2. You trace into the kernel code path triggered by application actions (syscalls/traps, memory faults, scheduling, filesystem calls).
3. You complete the missing kernel parts so the application(s) run correctly.

The repository targets Linux as the development environment, and execution typically uses the RISC-V emulator Spike.

---

## Lab 1: Traps (Syscalls), Exceptions, and Interrupts

### Lab1_1: Syscall/Trap Basics for `printu` (Implemented via `ecall`)

#### Experimental Requirements
- An application calls `printu(...)` instead of the standard `printf`.
- `printu` triggers a trap using the RISC-V `ecall` instruction.
- The kernel must handle the trap and provide the expected output/behavior (and support the required syscall flow).

#### Functions (Modules to Understand/Touch)
- `user/lab1_1_helloworld.c` (application entry)
- `user/do_print.c`
  - the inline assembly that issues `ecall` for trap invocation
- Kernel trap/syscall dispatch:
  - `kernel/strap/strap.c`
    - `handle_syscall(...)` (trapframe update + syscall return)
    - `smode_trap_handler(...)` case `CAUSE_USER_ECALL`
  - `kernel/syscall/syscall.c`
    - syscall implementations (at least syscall(s) required by `printu`)

#### What Needs to Be Implemented
- Implement the syscall handling path so that:
  - the trap handler recognizes user ecall (`CAUSE_USER_ECALL`),
  - syscall arguments are taken from the user trapframe registers,
  - the syscall return value is written back to `tf->regs.a0`,
  - the trap handler adjusts `tf->epc` so execution returns to the correct point after the ecall.
- Provide the specific syscall(s) needed by the `printu` runtime.

---

### Lab1_2: Machine-mode Trap Entry (M-mode)

#### Experimental Requirements
- Provide a machine-mode trap entry point (`mtrapvec`) so the system can correctly capture trap/interrupt events that occur in M mode.
- Save necessary registers/state for M-mode trap handling.
- Set up delegation so traps/exceptions are handled in S-mode kernel.

#### Functions (Modules to Understand/Touch)
- `kernel/arch/riscv/minit.c`
  - declares `mtrapvec`
  - saves an M-mode trap-frame pointer into `mscratch` (marker `@lab1_2`)
  - sets `mtvec` to `mtrapvec`
  - enables machine-mode interrupts (`MSTATUS_MIE`)
  - delegates interrupts/exceptions to supervisor via `mideleg`/`medeleg`
- `kernel/arch/riscv/strap_vector.S`
  - trap return path may rely on saved state prepared earlier
- `kernel/arch/riscv/mtrap.c` / assembly vector code (indirectly referenced by `mtrapvec`)

#### What Needs to Be Implemented
- Ensure the M-mode trap vector:
  - saves required state (into the `g_itrframe` area),
  - leads to correct supervisor-mode trap handling for supported causes.
- Ensure trap delegation is configured so supervisor mode receives the intended exceptions/interrupts.

---

### Lab1_3: Timer Interrupt Enablement and Timer Trap Handling (`ticks`)

#### Experimental Requirements
- Enable timer interrupts and handle timer traps in supervisor mode.
- Maintain a `ticks` counter and process timer events at each timer interrupt.

#### Functions (Modules to Understand/Touch)
- `include/kernel/config.h`
  - `TIMER_INTERVAL` (marker `@lab1_3`)
- `kernel/arch/riscv/minit.c`
  - `timerinit(...)` (enables M-mode timer interrupts and fires first timer irq)
- `kernel/strap/strap.c`
  - `g_ticks` and `handle_mtimer_trap()` (marker `@lab1_3`)
  - `smode_trap_handler(...)` case `CAUSE_MTIMER_S_TRAP`

#### What Needs to Be Implemented
- Configure timer interrupts so that supervisor can receive timer traps.
- Implement the timer trap handler behavior so that:
  - `g_ticks` is incremented,
  - per-process accounting fields (e.g., `CURRENT->tick_count`) are updated if required by later scheduling labs.

---

## Lab 2: Memory Management

### Lab2_1: Kernel/User Page Table Switching on Trap Return

#### Experimental Requirements
- During trap handling/return, the kernel must:
  - restore the kernel page table,
  - then switch to the user page table for user-mode execution.

#### Functions (Modules to Understand/Touch)
- `kernel/arch/riscv/strap_vector.S`
  - restores kernel page table from `p->trapframe->kernel_satp` (marker `@lab2_1`)
  - switches to user page table using `satp` with the user page table argument (marker `@lab2_1`)
- `kernel/proc/process.c`
  - `switch_to(...)` sets trapframe fields such as `kernel_satp`, `kernel_sp`, and `kernel_trap`

#### What Needs to Be Implemented
- Ensure trap return sequence performs correct `satp` switching:
  - kernel page table restored first,
  - then user page table enabled before resuming user execution.
- Ensure TLB is flushed appropriately (`sfence.vma`) as part of switching.

---

### Lab2_2: User Page Reclaim / Unmap Support (Triggered by a User Syscall)

#### Experimental Requirements
- Provide a syscall or mechanism to reclaim/free a user page given a virtual address.
- Correctly unmap the page (or update the process memory structures) and free underlying resources.

#### Functions (Modules to Understand/Touch)
- `kernel/syscall/syscall.c`
  - commented hint: “reclaim a page, indicated by `va`” (marker `@lab2_2`)
- Kernel memory manager modules (called by the reclaim syscall):
  - `kernel/mm/...` (e.g., unmap/free helpers)

#### What Needs to Be Implemented
- Implement the syscall behavior to:
  - locate the page mapping by user virtual address (`va`),
  - unmap from the process address space,
  - free backing memory / update refcounts if needed by later COW/reclaim logic.

---

### Lab2_3: User Page Fault Handling with Permissions and VMA Management

#### Experimental Requirements
- Implement a page fault handler that supports:
  - load page faults
  - store page faults
  - instruction fetch page faults
- Use VMA (Virtual Memory Area) information to:
  - determine the allowed permissions for the faulting address,
  - map in a page with the correct permissions if the access is valid,
  - reject invalid accesses.

#### Functions (Modules to Understand/Touch)
- `kernel/strap/strap.c`
  - `handle_user_page_fault(...)`
    - detects fault type (`CAUSE_LOAD_PAGE_FAULT`, `CAUSE_STORE_PAGE_FAULT`, `CAUSE_FETCH_PAGE_FAULT`)
    - sets `fault_prot` based on mcause
    - if `proc->mm` exists: calls `find_vma(proc->mm, addr)`
    - checks `(fault_prot & vma->vm_prot) == fault_prot`
    - aligns address and computes page index within the VMA
    - allocates and maps a new page using `mm_user_alloc_page(proc, page_va, vma->vm_prot)`
  - `smode_trap_handler(...)` cases for:
    - `CAUSE_STORE_PAGE_FAULT`
    - `CAUSE_LOAD_PAGE_FAULT`
    - `CAUSE_FETCH_PAGE_FAULT`
    - calling `handle_user_page_fault(...)`

#### What Needs to Be Implemented
- Complete the page fault path so that invalid accesses lead to failure (panic/abort as required by the lab behavior).
- Correctly map pages into the user page table:
  - correct page alignment,
  - correct permissions derived from VMA flags,
  - correct per-process memory structure usage.
- If the lab expects extensions (e.g., COW or stack auto-growth), implement them in the page fault handler where indicated by comments/hints.

---

## Lab 3: Processes

### Lab3_1: Process Pool Initialization and Basic Process Lifecycle Support

#### Experimental Requirements
- Initialize the process pool data structures.
- Support allocating a new process structure for user programs.
- Provide required per-process fields used by traps/scheduling/files.

#### Functions (Modules to Understand/Touch)
- `kernel/proc/process.c`
  - `init_proc_pool()` (marker `@lab3_1`)
    - initializes `procs[]` statuses and pids
  - `alloc_process()` (marker `@lab3_1`)
    - creates mm (`user_mm_create()`)
    - allocates kernel stack
    - allocates and sets up trapframe
    - initializes semaphores and file management (`init_proc_file_management()`)
- `kernel/strap/strap.c`
  - `switch_to(CURRENT)` which updates trapframe values for returning to user.

#### What Needs to Be Implemented
- Ensure `init_proc_pool` sets up the pool with consistent defaults.
- Ensure `alloc_process`:
  - creates a valid mm/address space object,
  - allocates kernel stack/trapframe,
  - initializes other fields required by subsequent labs (scheduling accounting, file system interface).

---

### Lab3_2: Implement `yield` Syscall

#### Experimental Requirements
- Implement the syscall of `yield`.
- `yield` must give up the processor and trigger rescheduling.

#### Functions (Modules to Understand/Touch)
- `kernel/syscall/syscall.c`
  - TODO hint for `sys_user_yield()` (marker `lab3_2`)
    - states the intended behavior in comments

#### What Needs to Be Implemented
- `sys_user_yield()` must:
  - change the currently running process status to `READY`,
  - insert the process into the rear of the ready queue,
  - schedule another READY process to run.

---

### Lab3_3: Round-Robin Scheduling via Timer Interrupts

#### Experimental Requirements
- Implement round-robin scheduling.
- On timer interrupts, update time slice usage and rotate to the next runnable process.

#### Functions (Modules to Understand/Touch)
- `kernel/strap/strap.c`
  - `rrsched()` (marker `@lab3_3`)
    - includes a detailed hint about how to update `tick_count`, status, ready queue, and schedule
  - `smode_trap_handler(...)` case `CAUSE_MTIMER_S_TRAP`
    - calls `rrsched()`
- `include/kernel/process.h` / scheduler structures
  - fields like `tick_count`, `TIME_SLICE_LEN`, ready queue interfaces

#### What Needs to Be Implemented
- Implement `rrsched()` as described by the hint:
  - increment accounting for `CURRENT`,
  - if consumed time slice (`tick_count >= TIME_SLICE_LEN`):
    - reset tick_count,
    - change CURRENT status to `READY`,
    - place CURRENT at the rear of the ready queue,
    - schedule a READY process to run.

---

## Lab 4: Device and File (VFS + RAM Disk, Directory and Links)

### Lab4_1: Filesystem Initialization (VFS + Device Integration)

#### Experimental Requirements
- Initialize the filesystem subsystem so user applications can use file-related syscalls.
- Build VFS interfaces and connect them to a device/filesystem implementation (the repo uses a RAM disk + VFS layer).

#### Functions (Modules to Understand/Touch)
- `kernel/kernel.c`
  - calls `fs_init()` (marker `@lab4_1`)
- `kernel/fs/...`
  - `vfs.c` (VFS utilities)
  - `ramdev.c`, `rfs.c` (RAM disk / ramfs-like FS)
  - `hostfs.c` (interface for host-fs)
- `kernel/proc/proc_file.c`
  - interface between file system and kernel/processes (marker `@lab4_1`)

#### What Needs to Be Implemented
- Ensure the following are connected and usable:
  - VFS initialization,
  - device registration (RAM disk),
  - inode/dentry/directory operations needed by higher-level syscalls.

---

### Lab4_2: Implement Reading a Directory Entry (`readdir`)

#### Experimental Requirements
- Implement logic to read a single directory entry at a given offset.
- Fill the output `dir` structure with:
  - `dir->name`
  - `dir->inum`
- Correctly return end-of-directory status when offset reaches the list end.

#### Functions (Modules to Understand/Touch)
- `kernel/fs/rfs.c`
  - `rfs_readdir(...)`
  - TODO `(lab4_2)` with a hint explaining exactly how to use `p_direntry` and fill `dir`.

#### What Needs to Be Implemented
- Implement the missing part in `rfs_readdir`:
  - use `offset` to locate the proper cached directory entry (`p_direntry`),
  - copy its `name` and `inum` into the returned `dir`,
  - increment `offset` on success and return 0,
  - return -1 when no more entries exist.

---

### Lab4_3: Implement Hard Link Creation (`link`)

#### Experimental Requirements
- Implement creation of a hard link for an existing file under a directory.

#### Functions (Modules to Understand/Touch)
- `kernel/fs/rfs.c`
  - `rfs_link(...)`
  - TODO `(lab4_3)` and a multi-step hint.

#### What Needs to Be Implemented
- `rfs_link(parent, sub_dentry, link_node)` must:
  1. Increase link count (`i_nlink`) of the file being hard-linked.
  2. Append a new directory entry (dentry) to the parent directory using `rfs_add_direntry`.
  3. Persist the inode metadata changes to disk using `rfs_write_back_vinode` (or the repo’s equivalent persistence helper).

---

## Optional Challenge/Debug-Enhancements (Only What Can Be Inferred)

The README indicates there are challenge labs. In this repository snapshot, explicit code markers we can infer include:

### Lab1 Challenge 1 / Challenge 2: ELF Symbol/Debug Information Support for Richer Debug Output

#### Experimental Requirements (Inferred from code markers)
- Extend ELF loading/debug information so that function names and/or debug lines can be resolved.
- This is typically used to support richer user-side debugging features (e.g., backtrace-like output).

#### Functions (Modules to Understand/Touch)
- `include/kernel/elf.h`
  - `load_debug_infomation(...)`, `locate_function_name(...)`
  - `load_debugline(...)`, `print_chars(...)`
- `kernel/syscall/syscall.c`
  - commented hint for `sys_user_print_backtrace(...)` that uses `locate_function_name(temp_pc)`

#### What Needs to Be Implemented
- Implement the ELF debug/symbol support functions and wire them into the syscall/service that requires them (if your lab asks for backtrace or debug-line printing).

---

