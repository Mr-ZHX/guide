# UCAS OS Labs (Project 1-6) - Experiment Descriptions (English)

This document is an English experiment description guide for all OS course projects under `OS/Project1` to `OS/Project6`. It contains only (1) experimental requirements, (2) key functions/modules to understand, and (3) what needs to be implemented. It intentionally excludes “completed report answers/results” style content.

## Common Experimental Environment

All projects are based on a RISC-V dual-core board (PYNQ Z2 with Rocket core). They require using the provided `BOOT.bin` and `RV_BOOT.bin` on an SD card. Build/run instructions are described in `OS/README.md` and each project’s `README`.

---

## Project 1: Bootloader

### Experimental Requirements

- Implement a bootloader that prints an informational message.
- Implement kernel loading from the SD card via SBI SD-card read.
- Implement a jump into the kernel entry point.
- Implement a working `createimage` tool that packages bootblock and kernel into the expected `image` format.

### Functions (Modules) to Understand

- `bootblock.S`
- `head.S` (kernel entry-side startup code)
- SBI SD-card read calling convention (register-based)
- `createimage.c` (ELF segment extraction and image layout)
- `riscv.lds` (linker symbols such as BSS boundaries)

### What Needs to Be Implemented

- `bootblock.S`: set up SBI SD read arguments (destination address, size/sector count, start sector), perform the read, then jump to the kernel entry.
- `head.S`: clear the `.bss` region using linker symbols (e.g. `__bss_start`, `__BSS_END__`), set stack pointer, and transfer control to `kernel.c`’s `main`.
- `createimage.c`: parse ELF program headers and copy each loadable segment into the output image at the correct aligned positions; write kernel size information to the location required by the bootblock.

---

## Project 2: Simple Kernel (Part I + Part II)

### Experimental Requirements

- Implement basic task scheduling and context switching in supervisor mode on a single core.
- Implement locking primitives with blocking behavior (spinlock and mutex-like lock).
- Implement timer interrupt handling, syscall handling, and blocking sleep.
- Implement futex-style wait/wakeup and binary semaphore syscalls.
- Implement priority-based scheduling.

### Functions (Modules) to Understand

- PCB design: kernel stack pointer, user stack pointer, `preempt_count`, status/type/cursor/pid fields.
- Context switching assembly: `switch_to` (save/restore callee-saved registers and update PCB saved stack pointer).
- Scheduler primitives: `do_scheduler()`, `do_block()`, `do_unblock()`, ready queue and blocked queues.
- Preemption and interrupt control: enabling/disabling preempt based on `preempt_count` and CSR state.
- Trap/exception and syscall dispatch flow: exception entry -> interrupt_helper -> handler -> exception return.
- Synchronization primitives: futex wait/wakeup and binary semaphore operations.
- Priority scheduling: PCB fields for priority and waiting-time accounting.

### What Needs to Be Implemented

- Part I (single-core, voluntary yield): implement PCB initialization, PCB stack initialization (initial register context), and `do_scheduler()` state transitions and queue updates; implement `switch_to` to persist/restore context.
- Part I (locking): implement spinlock acquire/release and mutex acquisition/release; implement blocking/unblocking based on wait queues.
- Part II (clock interrupt pipeline): implement timer queue scanning and timer expiry handling so that unblock callbacks trigger scheduling; implement correct screen refresh/reflush behavior if required by the project.
- Part II (syscalls and blocking sleep): implement syscall dispatch in the kernel trap handler; implement `sleep`-style blocking via adding the task to a sleep queue and unblocking it at timeout.
- Part II (futex and binary semaphore): implement futex wait/wakeup syscalls and binary semaphore `get/op` syscalls; ensure preemption/interrupt nesting is correct during these operations.
- Part II (priority scheduling): extend PCB with `priority` and `ready_tick`, update `ready_tick` when enqueuing tasks, and implement next-task selection using the composite priority/wait-time policy.

---

## Project 3: Interactive OS and Process Management

### Experimental Requirements

- Provide an interactive shell with process management commands.
- Implement process lifecycle required by `kill` and `waitpid`.
- Provide synchronization primitives for user threads/processes using futex + binary semaphore support.
- Implement kernel-backed mailbox IPC (producer-consumer).
- Support dual-core execution correctly, including core binding via MASK policy and correct inter-core interrupt (IPI) handling.

### Functions (Modules) to Understand

- Shell: screen usage layout, command parsing, command invocation.
- Kernel process management: `do_kill()`, `do_waitpid()`, and PCB state transitions (READY/RUNNING/BLOCKED/ZOMBIE/EXITED).
- Resource reclamation: kernel stack, PCB removal, queue removal, timers, and lock/semaphore bookkeeping.
- Synchronization in user space: condition wait/signal/broadcast and barrier wait/destroy built on kernel syscalls.
- Mailbox IPC: mailbox naming/id mapping, per-mailbox message storage, per-mailbox wait queues, and blocking/unblocking logic.
- Dual-core correctness: per-core `current_running`, MASK-based selection, and exception/interrupt correctness when IPI arrives.

### What Needs to Be Implemented

- Shell commands: implement `ps`, `reset`, `clear`, `exec tasknum`, `kill pid`, `taskset mask tasknum`, and `taskset -p mask pid`.
- Kernel `do_kill()`: unblock tasks waiting on relevant queues, release held binary semaphore locks (track binsem acquisitions per PCB), remove the PCB from scheduling queues, delete pending timers, mark the process as killed, and transition it to ZOMBIE so parent/init can reclaim.
- Kernel `do_waitpid()`: reclaim immediately if the child is already ZOMBIE; otherwise block the parent on the child’s wait queue; implement the required control-flow workaround so the parent can safely reclaim after the child unblocks it.
- User-space synchronization: implement condition-variable and barrier primitives using futex and binary semaphore syscalls as the underlying mechanism.
- Mailbox IPC: implement `do_mbox_send()` and `do_mbox_recv()` with correct blocking when buffers are insufficient and correct wakeups on successful send/receive.
- Dual-core scheduling: update scheduler/task selection to respect PCB MASK per core; ensure IPI/exception handling clears the right software interrupt source and does not corrupt shared structures.

---

## Project 4: Virtual Memory Management

### Experimental Requirements

- Implement virtual memory with RISC-V paging (kernel and user use different page sizes).
- Implement page fault handling with swap-in/out behavior using SD-backed storage.
- Implement page replacement using NRU (Not Recently Used).
- Maintain TLB correctness when switching address spaces and when page mappings change.

### Functions (Modules) to Understand

- Page table structure: PTE V bit semantics and A/D tracking; use of a software-defined PTE bit to indicate whether a page resides on SD.
- Page fault causes: missing page allocation, swapped-out page fetch from SD, and A/D bit update behavior.
- Physical frame management: per-frame valid/pin flags, and frame-to-pt mapping for the replacement algorithm.
- Address space switching: `satp` configuration and TLB flush (local and remote sfence.vma where needed).
- Swap/replacement pipeline: integrate swap operations with blocking behavior.

### What Needs to Be Implemented

- C-Core page management: build user/kernel page tables consistent with the project’s expected page sizes; implement page table entry preparation helpers; implement physical frame allocation and pinning behavior for frames involved in swap/page replacement.
- Page fault handler:
- Implement page fault handler case for unmapped pages: allocate a physical page frame and map it.
- Implement page fault handler case for swapped-out pages: load the page data from SD into memory and then map it.
- Implement page fault handler case for missing A/D permissions: update the PTE’s A/D bits based on fault cause.
- NRU page replacement: implement NRU victim selection over valid, non-pinned frames based on accessed/dirty attribute combinations; integrate swap-out when a victim page must be evicted.
- TLB maintenance: flush after `satp` switching and also flush per-page or flush-all when mappings change require it.

---

## Project 5: Device Driver (Network Driver)

### Experimental Requirements

- Implement a network (Ethernet) device driver using descriptor rings (TX ring and RX ring).
- Implement interrupt-driven handling for transmit completion and receive availability.
- Ensure receive correctness for multi-packet scenarios even when only a single interrupt is raised (wait until all expected packets arrive before copying to the user buffer).
- Provide syscalls for network send/receive and for enabling/disabling network IRQ mode.

### Functions (Modules) to Understand

- Descriptor ring details: used/new bits, address fields, and wrap-around behavior.
- Interrupt processing: identify whether the main completion is TX or RX by reading device status registers; keep descriptor state consistent with hardware status.
- Scheduling integration: block waiting tasks and unblock them once the required TX/RX completion is reached.
- Syscall layer: `sys_net_send`, `sys_net_recv`, and IRQ mode control.

### What Needs to Be Implemented

- Driver initialization: initialize TX/RX descriptors with correct initial flags and ring addresses.
- Send path: implement the kernel-side send so that each syscall triggers a single packet send; detect TX completion in the interrupt handler and unblock waiting sender tasks.
- Receive path: implement receive support for `num_packet` packets using RX descriptors; in the interrupt handler, continue processing until all required packets arrive, then unblock the waiting receiver.
- Interrupt handler: read hardware status registers, set internal TX/RX completion flags, and route descriptor-ring processing into correct completion paths.
- Syscalls: implement syscall handlers that coordinate descriptor processing and scheduler blocking/unblocking.

---

## Project 6: File System

### Experimental Requirements

- Implement a simple SD-card-backed file system in the kernel.
- Implement superblock + bitmap/inode/data structures and their initialization.
- Implement directory and file operations exposed via syscalls and shell commands (`mkfs`, `statfs`, `mkdir`, `rmdir`, `ls`, `cd`, `touch`, `cat`, `ln` for soft/hard).
- Implement user-level file descriptor handling (open/read/write/close) backed by kernel file operations.

### Functions (Modules) to Understand

- Disk I/O and caching: `disk.c` read/write primitives; inode/block allocation and cache management.
- File system core: implement superblock magic validation and initialization flow; implement `fs.c` path resolution and directory-level operations.
- File operations: implement `file.c` file descriptor-based read/write/cat/touch/fopen/fread/fwrite/fclose behavior; implement indirect block handling when file size exceeds direct pointers.
- Link semantics: hard link shares inode; soft link uses a separate inode but may reference target data blocks (follow the project’s non-Linux behavior noted in code/comments).

### What Needs to Be Implemented

- File system initialization (`mkfs`): validate and initialize superblock; clear block/inode maps; set up root directory metadata; initialize superblock counters correctly.
- Allocation/free: implement `alloc_inode`/`free_inode` and `alloc_block`/`free_block`, ensuring correct mapping between inode/block IDs and cached structures.
- Path parsing and directory operations: implement multi-level path lookup for `mkdir`, `rmdir`, `ls`, and `cd`; ensure `mkdir/rmdir` modifies the correct parent directory level.
- File operations:
- Implement `touch`: resolve directory path, allocate an inode, and add/update the directory entry.
- Implement `cat`: resolve path, locate inode data blocks, and copy required sectors into the user buffer.
- Implement open/read/write/close syscall behavior: manage file descriptors and handle indirect blocks when needed.
- Link operations: implement hard links by reusing target inode and updating required metadata; implement soft links (`ln -s`) by creating a soft-link inode and connecting it to target data blocks following the project’s design.
- Ensure shell commands map to corresponding syscalls and kernel handlers.

