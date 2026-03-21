# CS:APP Labs — Experiment Documentation (Code-Driven Overview)

This document summarizes the labs contained in this repository under `csapp-labs/`. It is written to help a reader **quickly understand what each lab is about**, **what functionality must be implemented**, and **what success should look like**.  
It intentionally avoids implementation guidance and does not include any “someone else’s report” content.

The labs included here are:
`Data Lab`, `Bomb Lab`, `Attack Lab`, `Cache Lab`, `Shell Lab`, `Malloc Lab`, and `Proxy Lab`.

---

## 1. Data Lab

### Goal
Modify the integer/float manipulation code so that a set of bit-level functions behave exactly as specified.

### What you need to do
1. Edit `data/bits.c` (the file you will hand in).
2. Ensure your functions in `bits.c` pass all correctness tests in `btest`.
3. Ensure your code follows the operator/style restrictions checked by the provided `dlc` tool.

### What functionality is required
`bits.c` defines the student-editable functions. In this repository snapshot, the functions include:
1. `bitXor(int x, int y)`
2. `tmin(void)`
3. `isTmax(int x)`
4. `allOddBits(int x)`
5. `negate(int x)`
6. `isAsciiDigit(int x)`
7. `conditional(int x, int y, int z)`
8. `isLessOrEqual(int x, int y)`
9. `logicalNeg(int x)`
10. `howManyBits(int x)`
11. `floatScale2(unsigned uf)`
12. `floatFloat2Int(unsigned uf)`
13. `floatPower2(int x)`

### How to verify success (expected outcomes)
1. `dlc` reports no compliance issues for `bits.c`.
2. Running `btest` shows all functions passing (no failing test cases).

### Success criteria
Your `bits.c` is accepted by both:
- the `dlc` compliance checker (coding/operator restrictions), and
- the `btest` correctness harness (millions of test cases).

---

## 2. Bomb Lab

### Goal
Defuse a staged “bomb” program by providing correct input lines for each phase.

### What you need to do
1. Run `bomb/bomb.c` as a standalone program (typically compiled into a `bomb` executable).
2. Provide **six** distinct inputs, one per phase, so that the bomb reports each phase as defused.

### What functionality is required
The program prints a welcome message and then, sequentially, reads a line of input and calls:
`phase_1`, `phase_2`, `phase_3`, `phase_4`, `phase_5`, and `phase_6`.

Each phase expects its input to satisfy a specific condition; the program only advances to the next phase if the current phase is defused.

### How to verify success (expected outcomes)
When each phase input is correct, the program outputs messages indicating that the phase is defused (e.g., “Phase 1 defused…”, “That’s number 2…”, “Halfway there!”, etc.).

### Success criteria
All required phases are defused in a single run, resulting in the program reaching the later phases without triggering the failure behavior.

---

## 3. Attack Lab

### Goal
Exploit two provided target binaries (one for code injection, one for return-oriented programming) so that they execute the intended “target” behavior.

### What you need to do
Use the provided lab instance files and your crafted inputs to:
1. Pass the code-injection phases using `ctarget`.
2. Pass the return-oriented programming phases using `rtarget`.

### What functionality is required
The repository includes:
- `ctarget`: Linux binary with a code-injection vulnerability (phases 1–3).
- `rtarget`: Linux binary with a return-oriented programming vulnerability (phases 4–5).
- `cookie.txt`: a 4-byte signature required for this lab instance.
- `farm.c`: a gadget-farm source code file used in this instance of `rtarget`.

Your crafted input must cause the target programs to transition into the correct success path. In the original lab structure, the cookie/signature is used to validate that the exploit reached the intended functionality.

### How to verify success (expected outcomes)
Successful phases are those where the target program’s behavior indicates the exploit reached the required “touch/target” action rather than failing or terminating early.

### Success criteria
You successfully complete all phases that correspond to the two binaries:
- injection-based phases against `ctarget`
- ROP-based phases against `rtarget`

---

## 4. Cache Lab

### Goal
1. Implement a cache simulator that reports hit/miss/eviction statistics for a given trace.
2. Implement an optimized matrix transpose routine that reduces cache misses under a specified cache model.

### What you need to do
You will modify and submit:
1. `cache/csim.c`: your cache simulator.
2. `cache/trans.c`: your matrix transpose functions.

### What functionality is required
1. **Cache simulation (`csim.c`)**
   - The simulator processes memory traces and computes:
     - `hits`
     - `misses`
     - `evictions`  
   - The autograder expects your simulator to output these statistics via the helper interface in `cache/cachelab.c` (notably `printSummary`).

2. **Matrix transpose (`trans.c`)**
   - Your transpose functions follow the required prototype:
     `void trans(int M, int N, int A[N][M], int B[M][N]);`
   - The evaluation counts cache misses on a model described in the file:
     - 1KB direct-mapped cache
     - 32-byte block size
   - A specific grading function is identified using the string:
     `Transpose submission`  
     (the driver uses this string to select the graded transpose submission function).

### How to verify success (expected outcomes)
From the cache lab handout (`cache/README`), a typical validation flow is:
1. Build: `make`
2. Check cache simulator correctness: `./test-csim`
3. Check transpose correctness/performance on multiple matrix sizes: `./test-trans ...`
4. Run full grading driver: `./driver.py`

### Success criteria
1. `csim` produces correct hit/miss/eviction statistics on the provided traces.
2. `trans.c` produces correct transposes and achieves good miss performance under the specified cache model.

---

## 5. Shell Lab

### Goal
Build a small command-line shell that supports job control and passes trace-driven tests.

### What you need to do
Implement the missing parts in `shell/tsh.c` (the repository’s shell skeleton).
The shell is tested by a trace-driven driver using the provided trace files (`trace*.txt`).

### What functionality is required
This shell must support:
1. Parsing and evaluating command lines.
2. Built-in commands (e.g., at least the `bg`/`fg` behavior and related job-control commands described by the shell skeleton).
3. Job control with these job states:
   - `FG` (foreground)
   - `BG` (background)
   - `ST` (stopped)
4. Correct signal handling for:
   - `SIGINT` (Ctrl-C) → forward to the foreground job
   - `SIGTSTP` (Ctrl-Z) → stop the foreground job
   - `SIGCHLD` → update job list when children terminate or stop

The top of `tsh.c` specifies required job-state transitions and indicates that at most one job can be in the foreground at any time.

Concretely, the skeleton marks these functions as the ones you are expected to implement: `eval`, `builtin_cmd`, `do_bgfg`, `waitfg`, `sigchld_handler`, `sigtstp_handler`, and `sigint_handler`.

### How to verify success (expected outcomes)
The shell lab uses the trace driver (`sdriver.pl`) and the provided trace files. If your job control and signal behavior are correct, the program’s output on each trace will match the reference output.

The `shell/Makefile` defines commands to run tests for each trace (e.g., `test01`, `test02`, …).

### Success criteria
All trace-based tests pass, demonstrating correct job control behavior (foreground/background execution, stop/resume via job commands, and correct interaction with Ctrl-C/Ctrl-Z).

---

## 6. Malloc Lab

### Goal
Implement a correct and efficient dynamic memory allocator that supports `malloc`, `free`, and `realloc`.

### What you need to do
Edit `malloc/mm.c`. This file is the student implementation that will be tested.

### What functionality is required
Your allocator must provide (as used by the driver):
1. `mm_init()`: initialize allocator state.
2. `mm_malloc(size_t size)`: allocate a block of at least `size` bytes (with required alignment guarantees).
3. `mm_free(void *ptr)`: release previously allocated blocks.
4. `mm_realloc(void *ptr, size_t size)`: resize an existing block while preserving required data semantics.

Correctness is evaluated by the provided trace-driven driver against a reference model:
- allocations must not overlap and must remain within the simulated heap,
- pointers returned must meet alignment requirements,
- realloc must preserve data as expected by the harness.

Performance is also part of grading. The driver computes utilization and throughput-related performance indices.

### How to verify success (expected outcomes)
The `malloc/mdriver.c` comments indicate the driver:
- runs a set of trace files,
- checks correctness (“valid”),
- measures utilization (“util”) and runtime to compute throughput and a performance index.

Typically, running `make` builds the driver and then `mdriver` evaluates your `mm.c` implementation over the traces.

### Success criteria
1. The trace driver reports correctness for all tested traces (no invalid allocations/reallocations).
2. The performance metrics (utilization/throughput) meet the expectations of the lab’s grading scheme.

---

## 7. Proxy Lab

### Goal
Implement a concurrent web proxy that supports caching, and passes an autograder that compares fetched content.

### What you need to do
Implement the proxy logic in `proxy/proxy.c` and ensure it behaves like a caching, concurrent HTTP proxy.

### What functionality is required
The proxy lab’s autograder logic is described in `proxy/driver.sh`. It tests the proxy in three dimensions:
1. **Basic correctness (Basic)**
   - The driver starts a Tiny web server and your proxy.
   - It downloads a list of files (text and binaries) through your proxy.
   - It compares the downloaded results with direct downloads from Tiny.

2. **Concurrency (Concurrency)**
   - The driver starts:
     - Tiny web server
     - your proxy
     - an additional blocking server (`nop-server.py`) that does not respond
   - The driver attempts to fetch content that should succeed even while one connection is blocked, validating that the proxy can handle concurrent connections without deadlocking or stalling all progress.

3. **Caching (Cache)**
   - The driver fetches a set of files through the proxy while Tiny is running.
   - Then it kills Tiny.
   - It requests one file again via the proxy.
   - Success requires that the proxy serves the cached copy so the fetched output still matches the original file.

The driver also enforces timeouts for network operations.

### How to verify success (expected outcomes)
Run the provided autograder script (`driver.sh`). If the proxy correctly implements request handling, concurrency, and caching behavior, the driver will report non-zero scores in Basic/Concurrency/Cache sections and a higher total score.

### Success criteria
1. Files fetched through the proxy match the files fetched directly from Tiny.
2. Fetches succeed under the concurrency stress scenario.
3. Cached responses remain correct after the origin server is terminated.

---

## Overall Checklist (Quick Reference)
- `data/bits.c` passes `dlc` and `btest`.
- `bomb` reaches later phases for correct inputs.
- `attack` successfully completes injection and ROP phases against the appropriate targets.
- `cache/csim.c` reports correct hit/miss/eviction statistics; `cache/trans.c` produces correct transposes with good miss behavior.
- `shell/tsh.c` passes all trace-driven job-control tests.
- `malloc/mm.c` passes trace-based correctness and achieves acceptable utilization/throughput.
- `proxy/proxy.c` passes the autograder’s Basic/Concurrency/Cache checks by matching downloaded outputs and demonstrating concurrency + caching.

