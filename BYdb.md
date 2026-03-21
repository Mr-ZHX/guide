# BUPT Database Labs — Overview (English)

This repository holds **database systems** coursework for **BUPT** (北京邮电大学, **2024–2025 Fall** *Database System Principles*), per root `README.md`.

The tree is organized as **`lab<id>-<topic>/`** with **`sql/`** scripts, optional **`src/`**, and **`run.sh`** runners. **`login.sh`** helps open a client session to the configured server.

This document lists **only labs that exist as folders in the repository** (`lab1-creation` … `lab7-transaction`). It states **what each lab is about** and **what artifacts you produce**, without SQL coding recipes or student report text.

**Engine note:** Scripts invoke **`gsql`** (openGauss-style CLI) with host/port/database/user set inside each `run.sh`. Adjust if your instructor uses another RDBMS.

---

## Lab 1 — Schema creation and loading (`lab1-creation/`)

**Theme (README item 1):** **Create tables** and **load data**.

**What the framework contains:** One SQL file per **TPC-H–like** entity (**`region`**, **`nation`**, **`customer`**, **`supplier`**, **`part`**, **`partsupp`**, **`orders`**, **`lineitem`**) plus **`constraints.sql`** for keys and referential integrity. **`run.sh`** runs a chosen script against the lab database.

**What you do:** Define the relational schema and constraints, import or generate base data so later labs can query the same **`buptlab_database`** (name as in scripts).

**Outcome:** All core tables exist, constraints apply, and data is present for subsequent experiments.

---

## Lab 2 — Queries and data changes (`lab2-crud/`)

**Theme (README item 2):** **Query** and **modify** data.

**What the framework contains:** Many numbered SQL tasks (e.g. `1.sql`, `2.1.sql`, `11-1.sql`, `23-2.sql`, …) covering **SELECT** (filters, ordering, aggregates, joins, etc.) and **DML** style work. **`run.sh <file-code>`** executes one script and can store **`result/`** output.

**What you do:** Write or complete SQL so each task meets the assignment’s expected result for the TPC-H-style dataset.

**Outcome:** Correct query results and updated table contents per task.

---

## Lab 3 — Integrity constraints (`lab3-constraints/`)

**Theme (README item 3):** **Integrity**—keys, checks, foreign keys, triggers.

**What the framework contains:** Scripts grouped by concern in filenames, for example:

- **Primary key** behavior (duplicates, NULLs, checks)
- **Referential** integrity (**RESTRICT**, **CASCADE**, **CHECK**-style semantics)
- **Entity** integrity: **NOT NULL**, **DEFAULT**, **CHECK** on columns
- **Functional dependency** exercise (`functional-try.sql`)
- **Triggers** (`trigger-*-add.sql` / `trigger-*-try.sql`) to enforce rules on DML events

**What you do:** Create and test constraints and triggers; observe acceptance vs rejection of bad rows.

**Outcome:** Database rejects invalid states and optionally runs trigger logic as specified.

---

## Lab 4 — Database programming interfaces (`lab4-api/`)

**Theme (README item 4):** **Application APIs** to the DBMS.

**What the framework contains:**

- **`jdbc/Main.java`** — Java client using JDBC (driver JARs expected under `jdbc/lib/`).
- **`odbc/main.c`** — C client using ODBC.
- **`run.sh`** builds/runs either **`odbc`** or **`jdbc`** path.

**What you do:** Implement or extend the sample programs to connect, run SQL or prepared statements, and process results per the course spec.

**Outcome:** Working small applications that talk to the same database as the SQL labs.

---

## Lab 5 — Physical design (`lab5-physics/`)

**Theme (README item 5):** **Physical** storage and tuning primitives.

**What the framework contains:** Scripts named by topic:

- **`1-tablespace.sql`** — tablespace layout
- **`2-partition.sql`** — partitioning
- **`3-index.sql`** — indexes
- **`4-segment.sql`** — segment-related physical options (as defined by the handout)

**What you do:** Apply physical DDL/features supported by the server and observe layout or performance implications.

**Outcome:** Tables/indexes/partitions configured according to each subtask.

---

## Lab 6 — Query optimization (`lab6-optimization/`)

**Theme (README item 6):** **Optimization** awareness.

**What the framework contains:** Scripts such as **`1-explain_usage.sql`**, **`2-view.sql`**, and many **`3-*`** files comparing plans or rewrite patterns (**composite index**, **join**, **subqueries**, **aggregates**, **DISTINCT**, **UNION**, **OR vs UNION**, **functions in predicates**, **small-table** heuristics, **redundant FROM**, etc.).

**What you do:** Use **`EXPLAIN`** (or equivalent) and schema changes to compare **costs/plans** and reason about optimizer behavior.

**Outcome:** Documented plan differences and correct use of indexes/views/rewrites as required.

---

## Lab 7 — Transactions (`lab7-transaction/`)

**Theme (README item 7):** **Transactions**, isolation, recovery.

**What the framework contains:**

- **`init.sql`** — setup tables (e.g. duplicated **`partsupp`** slices) and **CHECK** constraints for demos
- **Constraint / commit control:** check violations, **COMMIT/ROLLBACK**, schema vs data changes under transactions, **savepoints**, simulated **failure** scenarios
- **Isolation:** scripts for **read committed** vs **repeatable read** (and related behaviors)
- **`3-deadlock.sql`** — deadlock observation
- **Shell helpers:** **`4-1-backup.sh`**, **`4-2-recover.sh`** for backup/recovery practice
- **`run.sh`** may drive **two parallel `gsql`** sessions for concurrency exercises

**What you do:** Run interleaved sessions, observe locking, isolation anomalies (where allowed), deadlocks, and practice backup/restore flows.

**Outcome:** Correct understanding demonstrated through script outputs and reports required by the instructor.

---

## Summary (folders in this repo)

| Folder | Topic |
|--------|--------|
| `lab1-creation/` | DDL + data for benchmark-style schema |
| `lab2-crud/` | SQL queries and updates |
| `lab3-constraints/` | Keys, FK rules, checks, triggers |
| `lab4-api/` | JDBC + ODBC sample programs |
| `lab5-physics/` | Tablespace, partition, index, segment |
| `lab6-optimization/` | EXPLAIN and plan comparison tasks |
| `lab7-transaction/` | ACID, isolation, deadlock, backup/recovery |

---

*README also mentions “报告” (reports); those are not part of the code framework and are omitted here.*
