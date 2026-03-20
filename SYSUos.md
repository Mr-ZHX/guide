# SYSU Operating System (2025 Spring) — Total Experiment Description (English)

This document consolidates the course experiment requirements and **implementation-focused** functional requirements for:

- `Lab 1` to `Lab 9`

Notes:
- The document contains **deliverables/requirements** and **what you need to implement**.
- It does **not** include any pre-existing lab report results or analysis text from others.

---

## Lab 1 — Compile a Linux kernel and build a minimal OS (Experiment Description)

### 1) Experimental Requirements (Deliverables & Constraints)

Deadline: **2025-03-09 24:00**

Submission:
1. Convert your **Lab report** to **PDF**.
2. Name it: `lab1-<student_id>-<name>.pdf`.
3. Submit to: `os_sysu_lab@163.com`.

You must **independently complete five parts**:
1. Environment setup
2. Compile a Linux kernel
3. Boot the kernel in QEMU with **remote debugging enabled**
4. Create **initramfs**
5. Compile and boot **BusyBox**

Language/Platform/CPU:
- Any language (C/C++/Rust) is allowed.
- Any platform (Windows/Linux/macOS) is allowed.
- Any CPU architecture (ARM/Intel/RISC-V) is allowed.

### 2) Functional Requirements (What You Need to Implement)

Implement the full build-and-boot pipeline for a minimal i386 OS environment:
1. **Build environment**
   - Install Linux (tutorial assumes Ubuntu), toolchains (e.g., GCC, NASM), and utilities such as QEMU and GDB.
   - Enable debugging support for the kernel build.
2. **Linux kernel compilation (i386 / 32-bit)**
   - Download a kernel in the tutorial’s specified 5.10.x series.
   - Configure and compile the **i386** kernel.
   - Ensure kernel debug info is enabled.
3. **QEMU boot + remote debugging**
   - Start QEMU with parameters that allow GDB to attach (`-s -S` pattern).
   - In GDB: load symbols, set breakpoints (e.g., `start_kernel`), and trace execution.
4. **initramfs creation**
   - Create a minimal initramfs (e.g., “Hello World” binary or an init script).
   - Package it with `cpio` and boot using `-initrd`.
   - Demonstrate the expected initramfs execution.
5. **BusyBox compilation and boot**
   - Download and compile BusyBox for i386 with static/no-shared-libs configuration as required.
   - Build a BusyBox-based initramfs, boot it with QEMU, and demonstrate basic userland functionality (e.g., a working shell / `ls`).

### 3) Reporting/Evidence (No Pre-made Results)

In your PDF report, include:
- Your overall approach and why you chose each step.
- Screenshots for the five required parts:
  - environment setup
  - kernel build
  - QEMU + GDB debugging
  - initramfs creation
  - BusyBox boot

---

## Lab 2 — Real-mode to protected tasks: MBR, BIOS interrupts, and assembly assignments

### 1) Experimental Requirements (Deliverables & Constraints)

Deadline: **2025-03-23 24:00**

Submission:
1. Package the code for **3 tasks + 1 optional task** (if chosen) in a ZIP.
2. Name the ZIP: `lab2-<student_id>-<name>`.
3. Submit code to: `os_sysu_lab@163.com`.
4. Submit the Lab report PDF to: `http://inbox.weiyun.com/zPIW1se1`.

Language/Platform/CPU:
- Any language (C/C++/Rust) is allowed.
- Any platform (Windows/Linux/macOS) is allowed.
- Any CPU architecture (ARM/Intel/RISC-V) is allowed.

### 2) Functional Requirements (What You Need to Implement)

#### Task 1 — MBR boot code (16-bit registers)
1. **1.1 Reproduce “Example 1”**
   - After MBR is loaded/executed, output “Hello World”.
   - Provide screenshots and a short explanation.
2. **1.2 Print student ID at (12,12)**
   - Modify Example 1 so that student ID starts at coordinate **(12,12)**.
   - Student ID foreground/background colors must differ from the tutorial.
   - Provide screenshots and explain the change.

#### Task 2 — Real-mode interrupts (INT 10h / INT 16h)
1. **2.1 Cursor exploration**
   - Using `int 10h`, move the cursor to **(8,8)**.
   - Retrieve and output the cursor position.
   - Provide screenshots and explanation.
2. **2.2 Output student ID using real-mode interrupts**
   - Starting from (8,8), output your student ID using BIOS interrupts.
3. **2.3 Keyboard input + echo**
   - Explore `int 16h` and implement “any key input and echo”.

#### Task 3 — Assembly assignments (32-bit registers)
You must complete code in `assignment/student.asm` and validate with `make run`.
1. **3.1 Branch logic**: translate the given pseudo-code into assembly at `your_if`.
2. **3.2 Loop logic**: translate the given pseudo-code into assembly at `your_while`.
3. **3.3 Function implementation**:
   - Implement `your_function` to iterate through a null-terminated `string`
   - For each char, call `print_a_char` appropriately.

Provide screenshot of `make run` results and explain your implementation.

#### Task 4 (Optional) — Bonus MBR shooting program (<= 510 bytes)
- Shoot a character from **(2,0)** toward the **down-right 45-degree** direction.
- Bounce on boundaries; after each bounce continue with 45-degree direction determined by bounce location.
- Optional effects: color change, bidirectional shooting, etc.
- Total program size must be **<= 510 bytes**.
- Provide screenshots and a brief explanation.

---

## Lab 3 — From real mode to protected mode: bootloader, LBA/CHS, and protected mode entry

### 1) Experimental Requirements (Deliverables & Constraints)

Deadline: **2025-04-09 24:00**

Submission:
1. Package code for **2 tasks + 1 optional assignment** in a ZIP.
2. Name the ZIP: `lab3-<student_id>-<name>`.
3. Submit code to `os_sysu_lab@163.com`.
4. Submit Lab report PDF to: `http://inbox.weiyun.com/NuWl0loN`.

Materials:
- Example codes are provided under `src/`.

Language/Platform/CPU:
- Any language (C/C++/Rust) is allowed.
- Any platform (Windows/Linux/macOS) is allowed.
- Any CPU architecture (ARM/Intel/RISC-V) is allowed.

### 2) Functional Requirements (What You Need to Implement)

#### Task 1 — Bootloader disk I/O & LBA/CHS (Example 1)
1. **1.1 Reproduce “Example 1”**
   - Reproduce Example 1’s bootloader behavior.
   - Explain your approach and provide screenshots.
   - Optionally implement your own LBA-based disk access.
2. **1.2 Replace LBA28 reading with CHS reading**
   - Convert LBA28-based disk read into CHS-based read.
   - Provide the **LBA-to-CHS conversion formula** and explain it.

Key CHS parameters:
- DL: `80h`
- Sectors per track: `63`
- Heads per cylinder: `18`

#### Task 2 — Enter protected mode (Example 2) with debugging
- Reproduce Example 2.
- Use GDB (or other tools) to set breakpoints at **four key steps** of entering protected mode.
- Analyze code and registers at each step.
- Provide screenshots.

#### Task 3 (Optional) — Run 32-bit custom assembly after protected mode
- Adapt “Lab 2 – Assignment 4” into a **32-bit** assembly program.
- Execute after entering protected mode and provide validation/screenshots.

---

## Lab 4 — Interrupts in protected mode: mixed programming, PIC, and clock interrupts

### 1) Experimental Requirements (Deliverables & Constraints)

Deadline: **2024-04-20 24:00**

Submission:
1. Package code for **4 assignments** in a ZIP named `lab4-<student_id>-<name>`.
2. Submit to the course email.
3. Submit Lab report PDF to: `http://inbox.weiyun.com/NOKI03hf`.

Materials:
- Example codes are provided under `src/`.

Language/Platform/CPU:
- Any language (C/C++/Rust) is allowed.
- Any platform (Windows/Linux/macOS) is allowed.
- Any CPU architecture (ARM/Intel/RISC-V) is allowed.

### 2) Functional Requirements (What You Need to Implement)

This lab targets:
- C build/code organization via Makefile
- Mixed C/assembly programming
- Protected-mode interrupt handling
- 8259A PIC programming
- Clock/timer interrupt handling

#### Assignment 1 — Mixed programming fundamentals
1. Reproduce Example 1.
2. Explain:
   - C calling assembly functions
   - Assembly calling C/C++ functions
   - Role of `global`, `extern`
   - Why C++ functions use `extern "C"`
3. Build Example 1 using Makefile and provide screenshots + explanation.

#### Assignment 2 — Kernel output customization
1. Reproduce Example 2.
2. After `setup_kernel`, replace “Hello World” with your **student ID**.
3. Provide screenshots and explanation.

#### Assignment 3 — Interrupt handler customization
1. Reproduce Example 3.
2. Replace the default interrupt handler with your own handler.
3. Trigger the interrupt to verify.
4. Provide screenshots and explanation.

#### Assignment 4 — Clock interrupt handling (C/C++ only)
1. Reproduce Example 4.
2. Implement the clock interrupt handling flow using C/C++ together with:
   - `InterruptManager`
   - `STDIO`
   - your own helper classes
3. You must **not** implement the interrupt handler purely in assembly.
4. Demonstrate an effect such as a scrolling “marquee” on the first screen line with your student ID and English name.
5. Provide screenshots and explain design.

---

## Lab 5 — Kernel threads: variadic mechanism, printf, PCB-based threads, and scheduling

### 1) Experimental Requirements (Deliverables & Constraints)

Deadline: **2024-05-07 24:00**

Submission:
1. Package code for **3 assignments + optional 1** in a ZIP named `lab5-<student_id>-<name>`.
2. Submit to `os_sysu_lab@163.com`.
3. Submit Lab report PDF to: `http://inbox.weiyun.com/3CiJFwEn`.

Materials: Example code is under `src/`.

Language/Platform/CPU:
- Any language (C/C++/Rust) is allowed.
- Any platform (Windows/Linux/macOS) is allowed.
- Any CPU architecture (ARM/Intel/RISC-V) is allowed.

### 2) Functional Requirements (What You Need to Implement)

#### Assignment 1 — Implement `printf`
- Learn and use variadic argument mechanism.
- Implement `printf` (improve the provided one or implement your own).
- Provide screenshots and explain how you did it.

#### Assignment 2 — Implement kernel threads
1. Design a PCB (you may add attributes such as priority).
2. Implement threads based on your PCB design.
3. Demonstrate execution with screenshots and explanation.

#### Assignment 3 — “Secret” of context switching
1. Write multiple thread functions.
2. Use GDB to trace `c_time_interrupt_handler`, `asm_switch_thread`, etc.
3. Explain:
   - how a newly created thread starts running
   - how a running thread is preempted and later resumes from the interrupt point
4. Provide screenshots/evidence as required.

#### Assignment 4 — Scheduling algorithm implementation (FCFS mandatory)
- Modify scheduling algorithm from RR.
- **FCFS is mandatory**; other algorithms are optional.
- Provide test cases, screenshots, and explanation of correctness and basic behavior.

---

## Lab 6 — Concurrency and locks: SpinLock, Semaphore, and synchronization problems

### 1) Experimental Requirements (Deliverables & Constraints)

Deadline: **2024-05-18 24:00**

Submission:
1. Package code for **3 assignments** in a ZIP named `lab6-<student_id>-<name>`.
2. Submit to `os_sysu_lab@163.com`.
3. Submit Lab report PDF to: `http://inbox.weiyun.com/75p8UNN3`.

Materials: code under `src/`.

Language/Platform/CPU:
- Any language (C/C++/Rust) is allowed.
- Any platform (Windows/Linux/macOS) is allowed.
- Any CPU architecture (ARM/Intel/RISC-V) is allowed.

### 2) Functional Requirements (What You Need to Implement)

#### Assignment 1 — Reproduce SpinLock/Semaphore, then implement an alternative lock
1. Reproduce tutorial’s SpinLock and Semaphore design.
2. Use them to solve “disappearing cheese burgers”.
3. Provide screenshots and explanation.
4. Implement an alternative lock method that is not the same as the tutorial approach.
5. Implement your own atomic primitive in `asm_utils.asm` and modify `sync.cpp`.
6. Test and provide screenshots plus explanation.

#### Assignment 2 — Producer–Consumer
1. **2.1** Demonstrate race condition:
   - Implement producer–consumer with multiple threads using **no synchronization**, show incorrect behavior.
2. **2.2** Semaphore solution:
   - Use semaphores to fix synchronization and show the corrected behavior.
3. Provide screenshots and explain your setup.

#### Assignment 3 — Dining Philosophers
1. **3.1** Create 5-thread philosopher scenario.
2. Use semaphores to implement the textbook solution.
3. Provide a deadlock example (why it happens) and propose a deadlock solution.
4. Provide screenshots and explanation.
5. **3.2 (Optional)** Demonstrate an actual deadlock case with waiting time and propose/implement a deadlock mitigation method.

---

## Lab 7 — Memory management: BitMap/AddressPool, 2-level paging, and virtual memory

### 1) Experimental Requirements (Deliverables & Constraints)

Deadline: **2024-06-03 24:00**

Submission:
1. Package code for **3 assignments + optional 1** into ZIP named `lab7-<student_id>-<name>`.
2. Submit to `os_sysu_lab@163.com`.
3. Submit Lab report PDF to: `http://inbox.weiyun.com/i21y4NDD`.

Materials: code under `src/`.

Language/Platform/CPU:
- Any language (C/C++/Rust) is allowed.
- Any platform (Windows/Linux/macOS) is allowed.
- Any CPU architecture (ARM/Intel/RISC-V) is allowed.

### 2) Functional Requirements (What You Need to Implement)

#### Assignment 1 — Reproduce 2-level paging and virtual memory alloc/free
- Enable memory management in 2-level paging setup.
- Implement memory allocation and freeing under the VM address space.
- Provide screenshots and explain the process.

#### Assignment 2 — Implement dynamic partition algorithm (physical memory allocation)
- Implement a dynamic partition algorithm (e.g., first-fit/best-fit or your own).
- Integrate it into the physical page allocation logic.
- Provide tests and screenshots and explain.

#### Assignment 3 — Reproduce virtual page memory management + testing
- Analyze the 3-step allocation process and the virtual free process.
- Create tests to detect bugs; fix and re-test if bugs exist.
- If no bugs exist, validate correctness with test evidence and explanation.

#### Assignment 4 (Optional) — Page replacement algorithms
- Implement page replacement (FIFO/LRU/etc.) or your own algorithm.
- You may simulate in C++ or implement in the Ubuntu lab environment.
- Provide tests, screenshots, and analysis.

---

## Lab 8 — Privilege mode: system calls, process creation, and fork/wait/exit

### 1) Experimental Requirements (Deliverables & Constraints)

Deadline: **2024-06-15 24:00**

Submission:
1. Package code for **3 assignments** in a ZIP named `lab8-<student_id>-<name>`.
2. Submit to `os_sysu_lab@163.com`.
3. Submit Lab report PDF to: `http://inbox.weiyun.com/AQRPynaF`.

Materials: code under `src/`.

Language/Platform/CPU:
- Any language (C/C++/Rust) is allowed.
- Any platform (Windows/Linux/macOS) is allowed.
- Any CPU architecture (ARM/Intel/RISC-V) is allowed.

### 2) Functional Requirements (What You Need to Implement)

Core topics:
- Privilege separation (kernel mode vs user mode)
- System call mechanism using interrupts
- Process creation with paging-based isolation
- Process scheduling modifications
- Implement `fork`, `wait`, `exit` (and analyze their behavior)

#### Assignment 1 — Write a system call and invoke it from a process
- Implement a new system call.
- Invoke it from a process.
- Provide:
  - evidence of correct execution (screenshots + explanation)
  - stack change analysis
  - explanation of TSS role during system call execution

#### Assignment 2 — Fork mystery
- Implement `fork`.
- Answer:
  - basic idea of fork based on code logic and execution results
  - trace the child process from first scheduling until fork returns using GDB
  - explain how child returns 0 and parent returns child PID using code logic + GDB

#### Assignment 3 — Implement `wait` and `exit`, and analyze exit/wait/ zombies
- Implement `wait` and `exit`.
- Provide analysis questions:
  - analyze exit execution process with code logic and examples
  - analyze why exit can be invoked implicitly and why exit return value is 0
  - analyze wait execution process
  - propose and validate an effective method to reclaim zombie processes if a parent exits before a child

---

## Lab 9 — Course Projects (Final): choose projects and implement them

### 1) Experimental Requirements (Deliverables & Constraints)

Deadline: **2025-07-04 24:00**

Submission:
1. Submit a ZIP containing code of the selected project(s) and the Lab report PDF.
2. ZIP name: `lab9-<student_id>-<name>`.
3. Submit code to `os_sysu_lab@163.com`.
4. Submit the large project report PDF to: `http://inbox.weiyun.com/BGduXTlA`.

Rules:
- Multiple projects can be implemented; scores can accumulate.
- Different projects’ code must be in different folders.
- You may implement on your own OS or on uCore/xv6.
- Copying without rework may cause zero score for the copied parts.

Language/Platform/CPU:
- Any language (C/C++/Rust) is allowed.
- Any platform (Windows/Linux/macOS) is allowed.
- Any CPU architecture (ARM/Intel/RISC-V) is allowed.

### 2) Functional Requirements (What You Need to Implement)

Choose at least one project among Project 1/2/3.

#### Project 1 — Implement `malloc/free` system calls
Implement system calls:
- `malloc(size)` allocates arbitrary byte memory
- `free(ptr)` releases arbitrary byte memory

Options:
1. **Own implementation (100 points)**:
   - design and implement your own malloc/free
   - provide test cases
   - explain correctness using test outputs
   - provide screenshots and detailed analysis
2. **Reference approach (70 points + up to 30 bonus)**:
   - reproduce/test/verify the given approach
   - (Bonus) make it thread/process safe via synchronization/mutual exclusion

#### Project 2 — Filesystem implementation
Implement a basic filesystem supporting commands:
- `open`, `read`, `write`, `close`
- `cd`, `pwd`, `ls`, `cat`, `echo`, `rm`

Options:
1. **Step-by-step (85 + 15)**:
   - simulate with a fixed-size file “disk” on Windows/macOS/Linux, implement core filesystem features (85)
   - migrate the simulated filesystem features to your OS (15)
2. **All-in-one (100)**:
   - implement filesystem directly on your OS (100)
   - provide tests, output-based correctness explanation, screenshots, and analysis

#### Project 3 — Alternative exploration (choose your direction)
Pick an exploration topic and implement:
1. Virtual memory improvement: swap-out + page fault swap-in using a page replacement algorithm
2. Shell/system exploration:
   - implement a shell supporting command parsing/execution
   - optionally load ELF programs and create processes (as your chosen plan)
   - note: disk I/O via IO ports is kernel-mode only; user mode requires system calls if needed
3. Copy-on-Write fork:
   - implement COW semantics to avoid eager physical page copying
   - share read-only pages; on write, allocate/copy on demand
   - provide tests and correctness analysis

