# BUPT Operating System Labs — Overview (English)

This repository contains materials for the **Operating Systems** course (北京邮电大学, **2024–2025** fall, instructor **Ye Wen / 叶文**, per root `README.md`). There are **six** programming-oriented labs.

This document is for readers unfamiliar with the assignment: **what each lab is about** and **what you are expected to produce or observe**. It is **not** an implementation guide. It does **not** copy student lab reports.

**Environment:** Experiments are intended to run on **Ubuntu 24.04** (see `README.md`).

---

## Lab 1 — Process creation and management (`lab1-process`)

### Task 2 — Multi-process “exam” simulation

**Goal:** Practice **process creation**, **process groups**, **waiting**, and **signal handling** in C++.

**What you build:** A small system with roles analogous to a **professor**, **teaching assistants**, and **students**, implemented with **lightweight processes** (`clone`) and related APIs. **Students** compute a deterministic task (multiplication of two large integers) using one of several allowed methods; **TAs** supervise; the **professor** sets the task. The program must **handle termination signals** (e.g. `SIGTERM`) gracefully.

**Outcome:** Correct orchestration of parent/child processes, stable output, and correct arithmetic results under the organizer’s scheduling.

### Task 3 — Observation of processes and tools

**Goal:** Learn to **inspect** and **control** processes with standard Unix tools.

**What you do:** Use shell scripts (`ps`, `kill`, `pstree`, `jobs`, `strace`, `ltrace`, `top`, `vmstat`, `sleep`, etc.) in prescribed scenarios. Companion **C++** programs (`kill_test`, `strace_test`, `ltrace_test`) support exercises that involve **signals** and **library/system tracing**.

**Outcome:** You can interpret process trees, resource usage, and tracing output for simple programs.

---

## Lab 2 — Thread creation, management, and communication (`lab2-thread`)

### Task 1 — Hospital queue simulation (multi-threaded)

**Goal:** Use **POSIX threads** (`pthread`) and **synchronization primitives** (e.g. **semaphores** and mutexes) to model a **concurrent system**.

**What you simulate:** A **fixed number of doctors** and **many patients** with a **waiting room** capacity; patients move through states (waiting, treatment, left). The program must coordinate access to shared structures and enforce capacity/treatment rules.

**Outcome:** A deterministic, thread-safe simulation with clear logging of state transitions.

### Task 2 — Shared “dorm room” resource

**Goal:** Model **shared resource use** with threads and mutexes.

**What you simulate:** Several **people** share a **room** with a **capacity limit**; threads randomly **study** or **request** the room, wait if full, and **respect timeouts** for waiting. The exercise uses **mutexes** (and related patterns) to protect shared state and **serialized** console output.

**Outcome:** No race-corrupted counters; plausible scheduling of who uses the room when.

---

## Lab 3 — Inter-process communication (`lab3-ipc`)

### Task 1 — Message queues

**Goal:** Use **System V message queues** for high-volume **parent–child** data transfer.

**What you implement:** A **writer** sends many large **fixed-size frames**; a **receiver** reads and verifies them. **Throughput** and timing may be reported.

**Outcome:** Correct, loss-free bulk transfer with performance metrics.

### Task 2 — Shared memory

**Goal:** Use **shared memory** (`shm`) plus **semaphores** for producer–consumer style synchronization.

**What you implement:** Similar **frame** transfer as in Task 1, but **through a shared segment** with explicit **P/V** (or equivalent) coordination.

**Outcome:** Correct data integrity across processes without message copying for every byte.

### Task 3 — Pipes

**Goal:** Use **pipes** for IPC.

**Subtasks:** **Unnamed pipe** between parent and child, and **named pipe (FIFO)** between processes—each transferring large **frames** reliably.

**Outcome:** Full-duplex or half-duplex behavior as specified; no data corruption.

### Task 4 — Signals

**Goal:** Coordinate **multiple child processes** with **Unix signals**.

**What you implement:** Handlers for **SIGUSR1**-style signals (numeric codes in the source), **parent** broadcasts or sequences signals to children, **wait** for completion, and orderly shutdown.

**Outcome:** Demonstrated understanding of **process groups**, **signal delivery**, and **handler** behavior.

---

## Lab 4 — Synchronization and mutual exclusion (`lab4-synmut`)

**Goal:** Deepen **classical synchronization** skills with **POSIX threads**.

**What is in this tree:** A **`problem3-hospital`** POSIX implementation—**patients**, **doctors**, and **waiting-room** constraints, similar in spirit to the hospital exercise in Lab 2 Task 1 but framed as a **sync/mutex** lab.

**Outcome:** Deadlock-free behavior; fair or correct treatment scheduling; **no** lost wakeups or inconsistent shared state.

---

## Lab 5 — Multi-core multithreading and performance (`lab5-multithreading`)

**Goal:** Compare **serial** vs **parallel** implementations of a **CPU-intensive** workload and study **scalability** and **contention**.

**What you implement:** Multiple **variants** of the same numerical/bitwise workload (naming such as `parallel_2`, `parallel_3`, with optional **mutex**, **CPU affinity**, **cache-line padding**). The **same logical computation** is written to run with different threading and locking strategies.

**What you measure:** **Build** scripts produce binaries; **`run.sh`** / **`test.sh`** run trials (often with **high `nice` priority**) and log **wall-clock** times for comparison.

**Outcome:** Empirical understanding of **speedup**, **false sharing**, **lock overhead**, and **affinity** effects on a multicore Linux machine.

---

## Lab 6 — Inspecting the process address space (`lab6-memory`)

**Goal:** Relate **C source** layout to **virtual memory** and (optionally) **physical addresses**.

**What you program:**

1. A **`main`** program that uses **threads**, allocates **large arrays on stack and heap**, places data in **`.rodata`**, **`.bss`**, and **initialized data**, and prints **addresses** of those objects.
2. A **utility** that reads **`/proc/<pid>/pagemap`** for a chosen process and resolves a **virtual address** to **page/frame** metadata and, when present, a **physical address** (for learning/debugging; requires appropriate permissions).

**Outcome:** You can explain **where** stack, heap, and segments live in the address space and how the kernel exposes **page** information to user space.

---

## Repository map (quick reference)

| Directory | Lab topic |
|-----------|-----------|
| `lab1-process/` | Processes, `clone`, signals, shell tools |
| `lab2-thread/` | `pthread` + semaphores/mutex; dorm simulation |
| `lab3-ipc/` | Message queues, shared memory, pipes, signals |
| `lab4-synmut/` | Hospital-style POSIX sync |
| `lab5-multithreading/` | Performance comparison of parallel variants |
| `lab6-memory/` | Address space + `pagemap` |

---

## Note on this checkout

Some folders may contain **reference** or **instructor-style** solutions; your course may require you to **replace** parts with your own work. **Lab 1** in this tree appears as **`task2`** and **`task3`** only—if your syllabus lists **task1**, obtain it from the course site.

---

*This overview is derived from `README.md` and source file purposes only, not from student lab reports.*
