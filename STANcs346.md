# RedBase — Lab / Milestone Overview (English)

This document helps someone unfamiliar with the project understand **what work this codebase is organized around**. It is based **only on repository structure**: `Makefile` libraries, public headers, the interactive shell (`redbase.cc`), utilities, and test harnesses.

It describes **subsystems and intended capabilities**, not implementation recipes. It does **not** reuse content from the `*_DOC` files in `src/` (those are third-party **student design write-ups** and are not treated as course specifications here).

---

## Important: how “labs” appear in *this* tree

- There are **no** directories named `lab01`, `lab02`, … and **no** explicit “Assignment *N*” spec files in this snapshot.
- The project follows the classic **RedBase-style layered DBMS** used in many database courses. The natural **milestones** in *this* codebase are therefore the **named components** (`PF`, `RM`, `IX`, `SM`, `QL`) **plus** the **parser** and **command-line tools**, as wired in `src/Makefile` and `src/redbase.cc`.
- The top-level `README` states this tree is a **Stanford CS346 (Spring 2011)–era** reference implementation placed online by its author; **honor-code** rules may apply if you use it alongside a live course. Your instructor’s handout remains the **source of truth** for grading.

---

## Architecture snapshot

`redbase.cc` constructs the subsystem managers in dependency order and opens a database before invoking the parser:

- `PF_Manager` → `RM_Manager` → `IX_Manager` → `SM_Manager` → `QL_Manager` → `RBparse(...)`.

That ordering reflects how **lower layers** support **higher layers**.

---

## Component milestones (what each part is for)

### 1. PF — Paged File / Buffer Manager (`libpf.a`)

**Role:** Lowest-level storage abstraction: **files composed of fixed-size pages**, with a **buffer pool** (pin/unpin, force to disk), hash-based page lookup, and optional **statistics** (`PF_STATS` / `pf_statistics`).

**Outcomes you would expect from this layer:** reliable **page-level read/write**, **cached** access to pages, and **error handling** for file/page operations (see `pf_error`, `PF_Manager`, `PF_FileHandle`, `PF_PageHandle` in `pf.h` and related sources).

**Evidence in tree:** `pf_test1.cc`, `pf_test2.cc`, `pf_test3.cc`.

---

### 2. RM — Record Manager (`librm.a`)

**Role:** Stores **variable-length records** inside PF pages using **slot** bookkeeping (see `bitmap` sources and `rm_*` modules). Exposes **RIDs**, **file handles**, **insert/update/delete**, and **file scans** over all records.

**Outcomes:** callers can treat a relation file as a **sequence of records** without managing raw page layout; scans and predicates can be tested in isolation (`predicate.cc` bridges to expression evaluation needs).

**Evidence:** `rm_test.cc`, multiple `rm_*_gtest.cc` files.

---

### 3. IX — Index Manager (`libix.a`)

**Role:** Implements **secondary indexes** on top of RM data, backed by a **B+-tree** structure (`btree_node.*`, `IX_IndexHandle`, `IX_IndexScan`).

**Outcomes:** **search**, **insert/delete** of index entries, and **ordered scans** over key ranges (equality/range style operations as exposed by the index-scan API).

**Evidence:** `ix_test.cc`, `ix_*_gtest.cc`, `btree_node_gtest.cc`.

---

### 4. SM — System / Catalog Manager (`libsm.a`)

**Role:** **Database lifecycle** and **metadata**: open/close database; **create/drop** tables and indexes; **load** bulk data; **help/print** schema and contents; catalog relations (`relcat` / `attrcat` pattern—see `catalog.h`, `sm_manager.cc`).

**Outcomes:** a coherent **catalog** so QL can resolve names, attributes, and index availability; utilities `dbcreate` / `dbdestroy` prepare an on-disk database directory for `redbase`.

**Evidence:** `sm_manager_gtest.cc`, use by `redbase.cc` and parser.

---

### 5. Parser + Interpreter (`libparser.a`)

**Role:** Lex/yacc (`scan.l`, `parse.y`) plus AST construction (`nodes.c`, `interp.c`) to turn interactive SQL-like input into calls into **SM** and **QL** (`RBparse` entry point used from `redbase.cc`).

**Outcomes:** users can issue **DDL/DML** through the `redbase` shell against a named database.

**Evidence:** `parser_test.cc`, `Parser.HowTo`, generated `parse.c` / `scan.c` in build.

---

### 6. QL — Query Language / Execution (`libql.a`)

**Role:** High-level **query execution**: `Select`, `Insert`, `Delete`, `Update` as declared in `ql.h`, including **aggregates / GROUP BY hooks** in the `Select` signature, **ORDER BY**, and composition via **iterators** (`iterator.h`, `file_scan`, `index_scan`, `projection`, `predicate`).

**Physical operators** present in sources include **nested-loop join**, **block nested-loop join**, **merge join**, and **sort** (see `nested_loop_join.*`, `nested_block_join.*`, `merge_join.*`, `sort_gtest.cc`).

**Outcomes:** ad-hoc **queries** over registered relations, using **sequential or index-based** access paths and **join algorithms** as appropriate to the planner logic embedded in `ql_manager.cc`.

**Evidence:** `ql_test.*` sample scripts exercised by `ql_test.tester`, `ql_manager_gtest.cc`, and related `*_gtest.cc` files.

---

## Utilities and global definitions

- **`dbcreate` / `dbdestroy`:** create/remove a database instance on disk for tests and interactive runs.
- **`redbase`:** interactive client shell (see usage string in `redbase.cc`).
- **`redbase.h`:** shared constants (name lengths, attribute limits), **return-code ranges** per subsystem (`PF`/`RM`/`IX`/`SM`/`QL`), **attribute types**, and comparison operators used across layers.

---

## Testing landscape (what validates “done”)

- **Legacy component tests:** `pf_test*`, `rm_test`, `ix_test`, `parser_test`.
- **GoogleTest suites:** many `*_gtest.cc` modules compiled into a `gtest` executable (`Makefile` target `test` runs selected tests).
- **QL integration-style scripts:** numbered `ql_test.N` files driven by `ql_test.tester` (csh) to run query files against `dbcreate`/`redbase`/`dbdestroy`.

Together, these encode the **functional expectations** of each layer in code rather than in a separate lab PDF in this repository.

---

## Summary

**`redbase`** in this workspace is **not** split into numbered lab folders; it is split into **RedBase-style components** (**PF → RM → IX → SM → QL**) with a **parser** and **CLI**. Treat each component above as the unit of “what to build / what the layer must provide,” and rely on your **course materials** for official lab numbering, file restrictions, and grading.
