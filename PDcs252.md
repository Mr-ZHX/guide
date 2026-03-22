# CS 252 — Lab Framework Overview (English)

This document describes **only what exists under `cs252/`** in this workspace: four lab source trees (`lab1-src-final`, `lab2-src`, `lab3-src-final`, `lab5-src`) plus a short root `README.md`. It is for readers who have never seen the course: **what each lab is**, **what software you build**, and **how the tree is organized**. It **does not** give implementation steps or reuse personal write-ups bundled with the snapshot.

**Note:** There is **no `lab4-*` folder** here; the numbering gap is only a fact about this clone, not a claim about the official course schedule.

**In-tree pointer:** Root `README.md` labels the sequence as: **Project 1 — Mymalloc**, **Project 2 — Shell script**, **Project 3 — Shell**, **Project 5 — Socket**.

---

## Lab 1 / Project 1 — `lab1-src-final/` (dynamic memory allocator)

**Goal:** Implement a **substitute `malloc` / `free`**-style allocator in C (`myMalloc.c` / `myMalloc.h`) for a fixed **arena**, with explicit **header layout**, **free lists**, **splitting / coalescing**, **fenceposts**, optional **canaries** for corruption detection, and compile-time parameters such as **`ARENA_SIZE`** and **`N_LISTS`**.

**What you produce:** A library object linked by a **test harness** (`testing.c`, `runtest.py`) and many **micro-benchmark C files** under `tests/testsrc/` (covering correctness for allocation patterns, coalescing, double-free, odd/even frees, large requests, locks, etc.), plus **example programs** under `examples/`.

**Supporting tooling:** `utils/` scripts (e.g. differential testing, random allocation scripts), `printing.*` for debug output.

**Success criterion:** Passing the course’s automated / scripted tests (`make test` runs `runtest.py` after building tests).

---

## Lab 2 / Project 2 — `lab2-src/` (shell scripting)

**Goal:** A **Bash** assignment: write **monitoring and utility scripts** that exercise process statistics, file I/O, and text scoring—without a compiled binary deliverable.

**What is in the tree:**

- **`monitor.sh`** — Skeleton for a **process monitor**: validates arguments, tracks a **PID**, samples **CPU** and (in the extended part) **memory** usage against **thresholds**, writes timestamped **report files** under `reports_dir/`, and enforces periodic sampling semantics (structure visible in the script header and functions).
- **`pwcheck.sh`** — **Password-strength scoring** from a file: length window, bonuses/penalties based on character classes and patterns, prints a numeric score (student completion present in this snapshot).
- **`script-consumer.sh`**, **`p1-test.sh`**, **`test.sh`**, **`test-pass`**, **`testfile`** — Supporting harness / grading-style helpers for the scripting tasks.

**Deliverable concept:** Executable shell scripts meeting the instructor’s interface and behavioral checks (often including required **git** logging hooks embedded at the top of starter files in this tree).

---

## Lab 3 / Project 3 — `lab3-src-final/` (Unix shell)

**Goal:** Build a **command-line shell** named **`shell`**, implemented largely in **C++**, with **`lex`/`yacc`** (`shell.l`, `shell.y`) driving parsing, plus **`shell.cc`**, **`command.cc`**, **`simpleCommand.*`** for command execution.

**Functional scope (inferred from `test-shell/` names and `Makefile`):** A Bourne-like subset including **pipes**, **input/output/error redirection**, **environment variables** (`setenv`, `printenv`, `unsetenv`, `$`-style expansion), **wildcards**, **tilde** expansion, **quotes** and escapes, **subshells**, **`source`**, and **job-related** behavior (e.g. zombie tests), with optional **line-editing mode** (`tty-raw-mode.c`, `read-line.c` when `EDIT_MODE_ON`).

**Testing:** `test-shell/README` explains comparing against real shells (`sh`/`csh`/`tcsh`), **TTY-aware prompting** (`isatty`), and running **`./testall`** or individual tests.

**Build:** `make` produces `shell` and can build with **sanitizers** (`sanitize` target).

---

## Lab 5 / Project 5 — `lab5-src/` (HTTP server & networking)

**Goal:** Implement a **multi-threaded HTTP server** (`myhttpd.cc`) that listens on a **TCP port**, serves **static files** from a **document root** (`http-root-dir/htdocs/`), can generate **Apache-style directory listings** (HTML snippets are embedded in the source), and dispatches **CGI** programs from `cgi-bin/`. The project also explores **dynamic loading** with **`dlopen`** (targets `use-dlopen`, shared objects for `cgi-bin`).

**Other components (from `Makefile`):** **`daytime-server`** (simple network demo), **`hello.cc` / `hello.so`**, **`jj-mod`** modules for CGI experiments, **pthread** mutex around shared server state, optional **SSL**-related includes in the main server file (full TLS scope depends on course handout).

**Runtime assets:** Bundled `icons/`, sample HTML/SVG, and numerous **CGI** samples under `http-root-dir/cgi-bin/` and `cgi-src/`.

---

## Cross-cutting notes

- **Git hooks:** Several `Makefile`s and scripts append to **`.local.git.out`** and auto-**commit/push**—these are **course infrastructure** in this snapshot; treat them as environment-specific, not conceptual requirements for understanding the labs.
- **Official specs** (exact points, late policies, hidden tests) are **not** in this repository; use your term’s CS 252 course page if it differs from this tree.

---

## What this overview omits

- **Line-by-line script logic** and **allocator algorithms** (that would be “how to implement”).
- **Grading rubrics** and **partner rules**.
- Any **student report PDFs**—none are indexed here as primary specs.
