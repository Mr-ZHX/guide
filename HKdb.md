# Database Experiments (MySQL) — Lab Overview

This repository accompanies **“Database System Principles — Practice with MySQL”** (华中科技大学), with exercises hosted on [EduCoder](https://www.educoder.net/paths/x7rzkcvp). Each top-level folder is one lab; files are **numbered stages** within that lab (per `README.md`).

This document explains **what each lab is about** and **what you are expected to produce**, in **black-box** terms. It is **not** a solution guide and **does not** reproduce student reports or graded answers.

**Typical environment:** MySQL Server, often schemas such as `finance1`, `TestDb`, `testdb1`, `hms1`, `fib`, etc., as implied by the starter scripts in each folder.

---

## Lab 1 — Database, tables, and integrity (`CREATE`)

**Theme:** Data definition with SQL.

**What you do:** Complete a sequence of DDL tasks: create databases, define base tables with appropriate **data types**, **primary keys**, **foreign keys**, and other **integrity constraints** as required by each stage.

**Outcome:** A schema that matches the specification for each numbered `.sql` file (six stages in this tree).

---

## Lab 2 — Altering structure and constraints (`ALTER`)

**Theme:** Schema evolution.

**What you do:** Use `ALTER TABLE` (and related DDL) to rename objects, add or change columns, adjust constraints, and otherwise modify existing tables without recreating the whole database—following the blanks or instructions in each stage (four stages).

**Outcome:** Correct post-migration schema behavior verified against the course tests.

---

## Lab 3 — Data retrieval (`SELECT`)

**Theme:** Query writing on a shared relational schema (commonly a **financial / customer–product** scenario).

**What you do:** Write **19** separate queries. Skills covered across the stages include **filtering**, **sorting**, **joins** (inner/outer as needed), **aggregation** (`GROUP BY`, `HAVING`), **subqueries**, **top-N** reporting, **date/time** handling, and multi-step reporting (e.g. pivot-style summaries by weekday).

**Outcome:** Each `.sql` file returns the result shape and content expected by the autograder for that stage.

---

## Lab 3 (supplement) — Advanced `SELECT` (“Select part 2”)

**Folder:** `MySQL - 数据查询(Select)之二` (parallel to the numbered lab sequence).

**What you do:** Six additional query problems on a **wealth-management / product** style dataset, including **ranking** (e.g. top sellers), **preference / segmentation** logic, **relational division–style** questions (“customers who bought all …”), and **similarity** comparisons between clients or products.

**Outcome:** One `.sql` file per business question, matching required output.

---

## Lab 4 — Insert, update, and delete (DML)

**Theme:** Populating and maintaining data.

**What you do:** Six stages of **`INSERT`**, **`UPDATE`**, and **`DELETE`** statements (sometimes combined with simple checks) on the given tables, respecting keys and business rules.

**Outcome:** Tables end in the state required by each exercise.

---

## Lab 5 — Views

**Theme:** Virtual tables for reuse and access control.

**What you do:**  
1. Define a **view** that joins several base tables to expose a denormalized “detail” projection (e.g. client + insurance holdings).  
2. Write queries **against the view** (e.g. aggregated summaries by customer).

**Outcome:** View exists with correct definition; dependent queries run correctly (two stages).

---

## Lab 6 — Stored procedures and transactions

**Theme:** Server-side procedural logic and transactional behavior.

**What you do (three stages, illustrative of this repo):**  
- Implement a **stored procedure** that fills a table with the first *m* **Fibonacci** numbers, in an **idempotent** way (safe to run multiple times).  
- Implement a procedure that **schedules night shifts** over a date range using **cursors** and control flow (hospital / staff schema).  
- Implement a **funds transfer** procedure with **input/output parameters**, enforcing **balance and account rules** inside a **transactional** procedure body.

**Outcome:** Procedures compile and behave as specified when invoked.

---

## Lab 7 — Triggers

**Theme:** Automatic integrity enforcement on data change.

**What you do:** Create a **`BEFORE INSERT`** trigger on a **property / holding** table that checks a **type** field and validates the referenced product id against the correct lookup table (**finance**, **insurance**, **fund**, etc.). On violation, **signal** an error with a clear message.

**Outcome:** Invalid inserts are rejected; valid inserts proceed.

---

## Lab 8 — User-defined functions

**Theme:** Reusable scalar logic in SQL.

**What you do:** Define a **stored function** (e.g. total **deposit** balance across **savings** cards for a given **client id**), subject to server settings such as **`log_bin_trust_function_creators`**. Then write a **query** that uses the function to list customers above a **balance threshold**, sorted by total.

**Outcome:** Function and query both satisfy the specification.

---

## Lab 9 — Security and authorization

**Theme:** Accounts and privileges.

**What you do:** Use **`CREATE USER`**, **`GRANT`** (including **column-level** privileges and **`WITH GRANT OPTION`** where required), and **`REVOKE`** to implement a small access-control scenario on shared tables (e.g. **client** contact fields vs. **bank_card** balances).

**Outcome:** Each user can perform only the allowed operations.

---

## Lab 10 — Concurrency and transaction isolation

**Theme:** Observing **isolation levels** and **anomalies**.

**What you do:** Run paired scripts (e.g. `*_1.sql` / `*_2.sql`) that **set session isolation** (such as **READ UNCOMMITTED**), start **transactions**, and interleave **reads/writes** with **timing** (`SLEEP`) to demonstrate **dirty reads**, **non-repeatable reads**, or **phantoms**, depending on the stage. Other stages use **default** isolation (e.g. **REPEATABLE READ**) on schemas like **airline ticketing** to compare first vs. second read in one transaction after another session updates data.

**Outcome:** Query results and behavior match the exercise’s concurrency narrative (multiple stages).

---

## Lab 11 — Backup, logs, and recovery from media failure

**Theme:** Physical/logical backup and restore.

**What you do:** Shell scripts invoking **`mysqldump`** (and related restore / replay steps) to **back up** a database, simulate **loss**, and **recover** using dumps and/or binlog-oriented procedures as specified per script (`1_1`, `1_2`, `2_1`, `2_2`).

**Outcome:** After running the prescribed sequence, the database is restored to the expected state.

---

## Lab 12 — Database design and implementation

**Theme:** From conceptual model to SQL schema.

**What you do:** Produce an **ER diagram** (or equivalent design artifact as required by the platform), derive a **relational schema** (tables, keys, foreign keys), and implement it in MySQL (**DDL** / forward-engineered script). The exact **application domain** is defined by the course assignment on EduCoder; students’ submitted URLs or narrative text in personal files should **not** be treated as the canonical spec.

**Outcome:** Design document link + consistent SQL implementation (files such as `.sql`, `.drawio`, or notes may appear in the folder).

---

## Lab 13 — Database application development (Java / JDBC)

**Theme:** Client programs using **JDBC**.

**What you do:** Seven small Java programs: load the **MySQL JDBC driver**, obtain **`Connection`**, run **`Statement` / `PreparedStatement`** queries and updates, process **`ResultSet`**, and implement **parameterized** operations (e.g. **delete** a bank card by client id and card number). Programs typically target the **`finance`** schema.

**Outcome:** Each `N.java` compiles and passes the course’s functional checks.

---

## Lab 14 — B+ tree index implementation (C++)

**Theme:** Internal index structure (similar in spirit to **BusTub**-style teaching projects).

**What you do:** Implement a **disk-oriented B+ tree** used as an index: **page** layout (`b_plus_tree_page`), **internal** and **leaf** pages, **search** (`GetValue`, `FindLeafPage`), **insertion**, **deletion**, and an **iterator** for ordered scans—cooperating with a **buffer pool** and **latching** as indicated in the skeleton.

**Outcome:** The C++ sources compile into the project’s test harness and pass structural and correctness tests for the index API (files such as `b_plus_tree.cpp`, `b_plus_tree_*_page.cpp`, `index_iterator.cpp`, `b_plus_tree_delete.cpp`).

---

## Folder `DB report`

Present in the repository layout but **no substantive files** were found in this checkout; if used, it would typically hold a written report separate from code—omit from grading automation unless your instructor says otherwise.

---

## How to use this overview

- For **exact** wording, data sets, and pass/fail rules, follow the **EduCoder** path linked in `README.md`.  
- Files in this repo are **reference or submitted solutions**; treat them as examples of file naming and scope, not as the official problem statement.

---

*Document generated from repository structure, file names, and non-solution-descriptive comments only.*
