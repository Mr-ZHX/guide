# Operating System Lab 2025 Spring — Lab1–Lab4

> **Lab pages:**  
> [Lab1 - Bootloader](https://feng-xu-oslab.github.io/oslab-book/lab/2025-01-24-lab1.html) ·  
> [Lab2 - System Calls](https://feng-xu-oslab.github.io/oslab-book/lab/2025-01-26-lab2.html) ·  
> [Lab3 - Process Management](https://feng-xu-oslab.github.io/oslab-book/lab/2025-02-16-lab3.html) ·  
> [Lab4 - Process Synchronization](https://feng-xu-oslab.github.io/oslab-book/lab/2025-03-05-lab4.html)

---

## 1. Overview and Links

| Resource | URL |
|----------|-----|
| **Lab book (main)** | https://feng-xu-oslab.github.io/oslab-book/ |
| **Submission** | http://cslabcms.nju.edu.cn/ |
| **Materials, environment, code** | https://box.nju.edu.cn/d/2335dd322c8c4b4383fd/ |
| **FAQ** | https://docs.qq.com/doc/DVmJhTEVramRiaENZ |

| Lab | Topic | Deadline (UTC+8) |
|-----|--------|------------------|
| Lab1 | Bootloader | 2025-03-11 0:00 |
| Lab2 | System Calls | 2025-04-01 0:00 |
| Lab3 | Process Management | 2025-04-29 0:00 |
| Lab4 | Process Synchronization | 2025-05-20 0:00 |

---

## 2. Lab1 — Bootloader

### 2.1 Objectives

- Understand OS boot (bootloader load, hardware init, kernel start).
- Learn interrupt handling and clock interrupt setup.
- Use the timer for time-based control.
- Load a user program from disk and understand the memory model.

### 2.2 Requirements

**Requirement 1 — Real mode, periodic “Hello World”**

- Configure the **8253/8254** timer so that a clock interrupt fires at a custom interval (e.g. &lt;1000 ms).
- In the clock interrupt handler, print `"Hello, World!"` on the terminal **about every 1 second** in real mode.

**Requirement 2 — Protected mode “Hello World”**

- Switch from **real mode** to **protected mode**.
- In protected mode, print `"Hello, World!"` **once** on the terminal.

**Requirement 3 — Load and run “Hello World” from disk in protected mode**

- Switch to protected mode, then load the “Hello World” program from **sector 1** of the disk to a fixed memory address and execute it so that it prints `"Hello, World!"`.

### 2.3 Background (summary)

- **First instruction after power-on:** BIOS loads the MBR (512 bytes, magic `0x55`, `0xaa` at end) to `0x7c00` and jumps to `CS:IP = 0x0000:0x7c00`.
- **8254 timer:** Software timer interrupt vector `0x1C`; IVT entries at `0x70` (offset), `0x72` (segment). For ~20 ms period: control `0x36`, ports `0x43`/`0x40`, counter ≈ 23863 (1193180 / (1/0.02) − 1).
- **Real mode:** 16-bit; physical address = segment×16 + offset; IVT at `0x0000–0x03ff`.
- **Protected mode:** 32-bit; GDT, segment descriptors, selectors; enable A20, load GDTR, set CR0.PE=1, long jump to 32-bit code.
- **VGA text:** 80×25; framebuffer base `0xb8000`; 2 bytes per character (ASCII + attribute).

### 2.4 Approach (outline)

| Task | Main steps |
|------|------------|
| Real-mode timer print | Set 8253/8254, hook int `0x1C`; in handler call BIOS to print; set SP. |
| Real → protected | Disable interrupts; write `0x92` for A20; `lgdt`; CR0.PE=1; `data32 ljmp` to 32-bit; init DS/ES/FS/GS/SS and ESP. |
| Load from disk | Use framework `readSec(void *dst, int offset)`; load sector 1 to agreed address; jump to entry (see `app/Makefile`). |

### 2.5 Code layout

```
lab1-STUID/
├── lab1/
│   ├── Makefile
│   ├── app/
│   │   ├── Makefile
│   │   └── app.s
│   ├── bootloader/
│   │   ├── Makefile, boot.c, boot.h
│   │   ├── start_1.s   # real-mode timer print
│   │   ├── start_2.s   # real→protected + protected Hello World
│   │   └── start_3.s   # load and run disk Hello World
│   └── utils/
│       └── genBoot.pl
└── report/
    └── 231220000.pdf
```

### 2.6 Submission

- Implement the three behaviors in `start_1.s`, `start_2.s`, `start_3.s`.
- Report: briefly describe the three parts and your reasoning (e.g. effect of stack pointer). Include your **email** in the report.

---

## 3. Lab2 — System Calls

### 3.1 Overview

- **Boot chain:** real mode → protected mode → bootloader loads kernel → kernel loads user program.
- **Privilege:** distinguish kernel and user mode; complete IDT, TSS, and interrupt handlers.
- **System calls:** implement library wrappers and kernel handlers for `getChar()`, `getStr()`, `printf()`, `sleep()`, `now()` to understand the full path from user to kernel.

### 3.2 Requirements

1. **Interrupt mechanism**  
   Implement/refine **IDT**, **TSS**, and interrupt handlers for user/kernel transitions.

2. **`printf()` and handler**  
   Implement the syscall handler (e.g. `sysWrite()`). Support full formatting: `%d`, `%x`, `%s`, `%c`.

3. **Blocking `sleep()`**  
   Use the clock from Lab1 to implement a blocking `sleep(unsigned int seconds)` (not as a syscall; e.g. via `setTimeFlag`/`getTimeFlag` and a loop).

4. **Keyboard → serial echo**  
   From keyboard input, echo characters to the (simulated) serial/terminal.

5. **`getChar()` and `getStr()`**  
   Implement syscall handlers so that `getChar()` returns one character and `getStr()` returns a string (no extra formatting required).

6. **`now()` syscall and library**  
   Read RTC; `now()` returns date/time components (e.g. hours, minutes, seconds, day, month, year) via a struct.

7. **Application:** every 1 second print current time, e.g.  
   `printf("RTC time: %d-%d-%d %d:%d:%d.\n", t->year, t->month, t->m_day, t->hour, t->minute, t->second);`

### 3.3 Background (summary)

- **Serial:** `-serial stdio` in QEMU; `initSerial()`, `putChar()` for debugging.
- **Startup:** init serial → `initIdt()` → `initIntr()` (8259A) → `initSeg()` (GDT, TSS) → `initVga()` → `initKeyTable()` → `loadUMain()` → `enterUserSpace()`.
- **Privilege:** CPL, DPL, RPL; ring0 (kernel) vs ring3 (user); IDT gate DPL for syscall allows user `int $0x80`.
- **IDT:** 256 entries; gate descriptor holds selector + offset; syscall uses vector `0x80`.
- **TSS:** holds SS0/ESP0 (and SS1/ESP1, SS2/ESP2) for stack switch on interrupt from user to kernel; update TSS.esp0 on process switch.
- **Syscall:** parameters in EAX (syscall number), EBX, ECX, EDX, ESI, EDI; return in EAX; `int $0x80`; kernel uses TrapFrame from stack.
- **RTC:** ports `0x70` (index), `0x71` (data); registers for seconds, minutes, hours, day, month, year; BCD conversion.

### 3.4 Approach (outline)

- **Keyboard echo:** In IDT set gate for IRQ1 (e.g. vector `0x21`); in `irqKeyboard()` → `keyboardHandle()` get scan code, then output via serial (or VGA).
- **Timer in protected mode:** Program 8253 (e.g. `TIMER_PORT`, `FREQ_8253`, `HZ`); bind vector `0x20` to timer handler in IDT.
- **`sleep()`:** e.g. 10 ms tick; kernel `timeFlag`; handler sets it; user loop: `setTimeFlag(0)` then `while (getTimeFlag()==0);` repeated for the desired seconds.
- **`printf()`:** Gate for `0x80` with user DPL; handler `sysPrint`/`sysWrite` writes to VGA at `0xb8000`, updates cursor, handles `\n` and scroll.
- **`getChar()` / `getStr()`:** Syscall returns one char (e.g. in EAX) or fills user buffer; kernel uses segment override to read/write user memory.
- **`now()`:** Kernel reads CMOS RTC, fills struct; copy to user via syscall.

### 3.5 Code layout

```
lab2-STUID/
├── lab/
│   ├── app/main.c
│   ├── bootloader/
│   ├── kernel/
│   │   ├── include/
│   │   ├── kernel/   # doIrq.S, idt.c, irqHandle.c, timer.c, kvm.c, ...
│   │   └── main.c
│   ├── lib/         # lib.h, syscall.c
│   └── utils/
└── report/
```

### 3.6 Submission

- Submit buildable source and report; run `make clean` before packaging.
- Deliverables must implement `printf()`, `getChar()`, `getStr()`, `sleep()`, `now()`.

---

## 4. Lab3 — Process Management

### 4.1 Objectives

- **Process lifecycle:** create, run, wait, exit; **fork–exec** model; scheduling and context switch; syscall implementation.
- **Scheduling:** time-slice round-robin; process states (ready, running, blocked); idle process; non-blocking `sleep` support.

### 4.2 Requirements

1. **Idle process**  
   Wrap the end of kernel init as the **idle** process (PID=0). When no user process is ready, run idle.

2. **Process creation**  
   - **`fork`:** copy parent’s resources to a new process.  
   - **`exec`:** load and run a new program (replace image).  
   - **Fork–exec pattern:** idle forks; child does `exec` to run the application.

3. **Syscalls**  
   - **Non-blocking `sleep`:** calling process goes to a wait queue; kernel schedules others (or idle).  
   - **`getpid`:** return current process PID.

4. **Optional:** implement a **`wait()`**-style library and corresponding kernel support (e.g. `getppid`).

### 4.3 TDD

User program in `lab3/app/main.c` uses `fork`, `exit`, `sleep`, `getpid`. First add the library wrappers in `lab3/lib/syscall.c` that invoke the kernel via syscalls; then implement the kernel handlers.

### 4.4 Background (summary)

- **ELF load:** Program Headers type LOAD; use `off`, `vaddr`, `filesz`, `memsz` to load and zero BSS.
- **Enter user:** `enterUserSpace(entry)` builds iret frame (SS, ESP, EFLAGS, CS, EIP and segment regs) then `iret`.
- **Timer:** Same 8253 setup as Lab2; vector `0x20`; in Lab3 `timerHandle` drives scheduling.
- **Memory:** Kernel at `0x100000`; each user process in a fixed region (e.g. `pcb[i]` at `(i+1)*0x100000`, size `0x100000`); switch by changing segment bases.
- **PCB:** e.g. `ProcessTable`: kernel stack, `StackFrame` (regs), `stackTop`, `prevStackTop`, `state`, `timeCount`, `sleepTime`, `pid`, name. States: RUNNABLE, RUNNING, BLOCKED, DEAD.
- **Switch:** On timer (or sleep/exit), save context on current kernel stack; choose next RUNNABLE (round-robin); set TSS.esp0 to new process kernel stack; restore from new stack and `iret`.
- **Idle:** PID 0; if no one else is RUNNABLE, schedule to idle; it may `hlt` (e.g. `waitForInterrupt()`). In the handout, idle is set up to run in user mode and first calls `fork` then either `exec` (child) or loops (parent).

### 4.5 Syscalls to implement

- **`sysFork`:** Allocate a free PCB; copy parent memory and PCB fields; child returns 0, parent returns child PID; on failure return -1.
- **`sysExec`:** Replace current process image with the new ELF (load from disk), set entry and run.
- **`sysSleep`:** Set `sleepTime`, set state BLOCKED, call `schedule()`.
- **`sysExit`:** Set state DEAD; trigger scheduling (e.g. as in timer path).
- **`sysGetPid`:** Return current `pid`.

### 4.6 Critical sections

With interrupts enabled during syscalls, timer can interrupt and switch processes; shared data (e.g. VGA, globals) must be protected. Document in the report how you reason about critical sections.

### 4.7 Code layout

Same general structure as Lab2; under `kernel/kernel/`: `irqHandle.c` (e.g. `schedule()`, `timerHandle`, syscall handlers), `kvm.c` (e.g. `initIdle`, GDT, load user), `doIrq.S`.

### 4.8 Submission

- Buildable source and report; `make clean` before submit.
- Must implement `fork()`, `exit()`, `getpid()`, `sleep()` (and `exec` in kernel). Optional: `wait()` and `getppid()`.

---

## 5. Lab4 — Process Synchronization

### 5.1 Requirements

1. **Formatted input**  
   Implement a simplified **`scanf()`** (e.g. `%c`, `%6s`, `%d`, `%x`) and its kernel handler so that user input matches the format string. Example: input `Test a Test oslab 2025 0xadc` → output `Ret: 4; a, oslab, 2025, adc.`

2. **Semaphore syscalls**  
   Implement **SEM_INIT**, **SEM_POST**, **SEM_WAIT**, **SEM_DESTROY** and test with the given parent–child semaphore program (init with value 2; child waits, parent sleeps and posts).

3. **Shared variables**  
   Add syscalls and library: **createSharedVariable**, **destroySharedVariable**, **readSharedVariable**, **writeSharedVariable**; test with fork and read/write of a shared variable.

4. **Classic synchronization problems**  
   Use semaphores (and shared variables where needed) to implement and test:  
   - **Producer–consumer:** e.g. 4 producers, 1 consumer; print “Producer i: produce” / “Consumer: consume”; insert `sleep(128)` between P/V and produce/consume.  
   - **Reader–writer:** multiple readers, writers; implement at least one of reader-priority or fair (no starvation).  
   - **Dining philosophers:** 5 philosophers; print “think” / “eat”; P/V and think/eat with `sleep(128)`.  
   (Producer–consumer **or** dining philosophers is enough; reader–writer uses shared variables.)

5. **Optional:** **Spinlock** using **XCHG** to protect semaphore P/V and queue updates in the kernel (single-core; still need to consider interrupt-driven reentrancy).

### 5.2 Background (summary)

- **Semaphore:** integer `value`; P: decrement, if &lt;0 block on queue; V: increment, if ≤0 wake one. Init: mutex with value 1; condition sync with value 0.
- **Device / stdin:** A “Device” structure (like a semaphore) is used for keyboard: data goes to a buffer; reader blocks on the device until input is available; then `sysReadStdIn` reads from buffer into user space.
- **Shared variable:** Kernel keeps an array of shared variables (e.g. `SharedVariable` with state and value); create returns a descriptor; read/write by descriptor.
- **ListHead:** Doubly linked list for blocked queues (e.g. `sem[i].pcb`, `pcb[].blocked`).
- **Reader–writer:** WriteMutex for writer; CountMutex and Rcount for readers; first reader P(WriteMutex), last reader V(WriteMutex).
- **Dining philosophers:** e.g. 5 forks (semaphores); odd/even philosophers take left-then-right or right-then-left to avoid deadlock.

### 5.3 Implementation notes

- **`sysReadStdIn`:** If `dev[STD_IN].value == 0`, block current process on `dev[STD_IN]`. When woken (by keyboard handler), read from `keyBuffer` into user buffer; at most one process blocked on STD_IN.
- **`keyboardHandle`:** Put key code into `keyBuffer`; wake one process blocked on `dev[STD_IN]`.
- **Semaphore handlers:** `sysSemInit` (find free slot, set value), `sysSemWait` (P, block if needed), `sysSemPost` (V, wake one), `sysSemDestroy` (mark unused; no one should be blocked).
- **Shared variable:** Implement create/destroy/read/write in kernel and expose via syscalls and lib.

### 5.4 Code layout

Same tree as Lab3; new/updated: `Semaphore`, `Device`, `SharedVariable` arrays; PCB has `blocked` (ListHead); `sysWrite` branches to `sysWriteStdOut`; `sysRead` and `sysReadStdIn` for stdin; semaphore and shared-var syscalls.

### 5.5 Submission

- Buildable source and report; `make clean` before submit.
- Report must include your **email**.
- After Lab4.4, remove the extra `exit()` in the Lab4.3 parent so that all tests (scanf, semaphore, shared variable, sync problems) run in one `uEntry()`.

---

## 6. General Notes

- All deadlines are **UTC+8**; submit early to avoid last-minute issues.
- Use the **box** materials and recommended environment (e.g. GCC version) to avoid “boot block too large” or other environment-specific failures.
- Submit only via **http://cslabcms.nju.edu.cn/**.
- For each lab, read the Makefile to see which targets build which image (e.g. Lab1’s three different boots).
- Report content: explain your design and key code; include required sections (e.g. IDT/TSS for Lab2, critical sections for Lab3); state your email where required.

---

*This document is compiled from the official lab pages. For the latest details and clarifications, refer to [oslab-book](https://feng-xu-oslab.github.io/oslab-book/) and the course announcements.*
