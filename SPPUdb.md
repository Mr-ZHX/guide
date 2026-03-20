# SPPU-COMP TE Practicals (LP-1 & DBMS) - Experiment Descriptions (English)

This document is an English experiment description guide for the repository `SPPU-COMP-TE-PRACTICALS-LP1-AND-DBMS`. It summarizes what the practicals require you to build, the key functions/modules/algorithms you should understand, and what specifically needs to be implemented. It intentionally avoids copying “completed report answers/results” text from the repository files.

## Scope Overview

The repository is organized into three main practical groups:

- `spos/`: assembler/macro processing, dynamic linking via JNI, CPU scheduling, memory placement strategies, and page replacement algorithms.
- `Distributed System/`: socket programming, RPC (XML-RPC), election algorithms, and clock synchronization.
- `DBMS/`: SQL (DDL/DML/Joins/Connectivity), PL/SQL (triggers/cursors/stored procedures/functions), and MongoDB operations (CRUD/indexing/aggregation/map-reduce).

---

## SPOS (Systems/Programming) Practical A: Assembler & Macro

### Practical A1: Two-Pass Assembler

#### Experimental Requirements

- Implement the first pass of an assembler to build:
  - A symbol table for labels.
  - A literal table for literal constants.
  - A pool table for literal pools (e.g., handling `LTORG` and `END`).
  - An intermediate representation (intermediate code) based on the input assembly.
- Implement the second pass to translate intermediate code into final machine code using:
  - Symbol table addresses.
  - Literal table addresses.
- Support assembler directives and instruction formats used in the provided pseudo-assembly (e.g., `START`, `ORIGIN`, `EQU`, `DS`, `DC`, `LTORG/END`, and register-based instructions).

#### Functions / Modules to Understand

- `Pass_1` logic:
  - Initialize instruction set mapping (mnemonic -> opcode).
  - Maintain and update `SYMTAB`, `LITTAB`, and `POOLTAB`.
  - Generate `IC` (intermediate code) from the assembly input.
  - Resolve address expressions for `ORIGIN` / `EQU` using symbol addresses and arithmetic offsets.
  - Handle literal pool emission at `LTORG` and final emission at `END`.
- `Pass_2` logic:
  - Read tables produced by pass 1.
  - Parse intermediate code line by line.
  - Convert intermediate statements into final output with resolved addresses.

#### What Needs to Be Implemented

- Implement Pass 1:
  - Parse each assembly line, detect labels, directives, and instructions.
  - Update `LC` (location counter) according to directives (`DS` and `DC`).
  - Add symbols to `SYMTAB` with correct addresses.
  - Add literals to `LITTAB` and set their addresses when pools are emitted.
  - Emit intermediate code tuples for:
    - Imperative statements (`IS, opcode`-style records).
    - Declarative statements for `DS/DC`-style records.
    - Literal references (`L, literal_index`).
  - Output artifacts required by Pass 2 (e.g., `IC`, `SYMTAB`, `LITTAB`, `POOLTAB` files).
- Implement Pass 2:
  - Load `SYMTAB` and `LITTAB`.
  - Translate intermediate code into final numeric code format.
  - Resolve:
    - Symbol references (`S, index`) using symbol addresses.
    - Literal references (`L, index`) using literal addresses.
  - Ensure consistent formatting and line handling for directives that are not emitted as machine instructions.

---

### Practical A2: Macro Processing (Pass 1 & Pass 2)

#### Experimental Requirements

- Implement macro processing using two passes:
  - Macro Pass 1: build macro-related tables and produce an intermediate representation with macro definitions extracted.
  - Macro Pass 2: expand macro invocations by using macro tables to substitute parameters into the macro body.
- Support `MACRO ... MEND`-style definitions.
- Support parameter handling:
  - Positional parameters.
  - Keyword parameters (the practical uses keyword-style parameter entries via `KP` / `KPDT`-like mechanisms).

#### Functions / Modules to Understand

- `MacroPass1`:
  - Construct MDT (Macro Definition Table) and MNT (Macro Name Table).
  - Maintain parameter tables (e.g., `PNTAB` and keyword parameter descriptor tables).
  - Convert macro definitions into a stored MDT form where parameter placeholders are recorded.
  - Create/emit an intermediate file where macro calls are kept for later expansion.
- `MacroPass2`:
  - Read `MNT`, `MDT`, and parameter tables generated in Pass 1.
  - For each macro invocation in the intermediate code:
    - Bind actual arguments to formal parameters.
    - Output expanded instructions/records into `pass2.txt` (or the project’s final expanded output).

#### What Needs to Be Implemented

- Implement Macro Pass 1:
  - Detect macro start (`MACRO`) and store macro name and parameter metadata in MNT.
  - Parse parameter lists and record mappings for:
    - Positional parameters.
    - Keyword parameters and their storage in a keyword parameter descriptor table.
  - Build MDT by storing macro body lines, replacing parameter tokens with `P, index`-style placeholders.
  - Record the correct pointers/indices for where each macro’s MDT body begins and ends.
  - Preserve non-macro lines into an intermediate representation for Pass 2.
- Implement Macro Pass 2:
  - Parse intermediate code and detect macro invocation tokens that match entries in MNT.
  - For each macro invocation:
    - Build an argument binding table (formal parameter index -> actual argument token).
    - Walk through the macro body in MDT until `MEND`, emitting expanded output lines.
  - Preserve lines that are not macro invocations.

---

### Practical A3: Dynamic Link Library via JNI

#### Experimental Requirements

- Implement a JNI-based bridge where Java calls native C functions implemented in a shared library (DLL/shared object).
- Provide arithmetic operations as native methods and ensure correct type mapping:
  - Integer operations (add/subtract/multiply/divide-by-integer style methods).
  - Floating-point operations (division and square root behavior).
  - Integer modulo operation.
  - Power function.
  - Sqrt function.
- Load the native library from the Java side and provide a user-interactive main entry.

#### Functions / Modules to Understand

- `JNI.java`:
  - Declare `native` methods that correspond to C JNI-export functions.
  - Load the native shared library via `System.load(...)`.
  - Provide a `main` that reads input and calls all native arithmetic operations.
- `JNI.c`:
  - Implement JNIEXPORT functions matching Java method signatures.
  - Use C math library for functions like `pow` and `sqrt`.
  - Print or return computed results (the practical prints results).

#### What Needs to Be Implemented

- Implement native methods in C:
  - Create JNI-export functions with the correct mangled name/signature mapping.
  - Perform requested arithmetic using C operators and math library calls.
  - Output results in a consistent format (or return them if the project’s JNI design requires return values).
- Implement the Java side to:
  - Provide matching native method declarations.
  - Correctly load the native library.
  - Collect user input and call all native operations.

---

## SPOS Practical B: CPU Scheduling, Memory Placement, Page Replacement

### Practical B1: CPU Scheduling

#### Experimental Requirements

- Implement CPU scheduling simulations for multiple scheduling policies.
- For each policy:
  - Accept from user a set of processes with at least `arrival time` and `burst time` (and `priority` for priority scheduling).
  - Simulate preemptive or non-preemptive execution as required by the policy.
  - Compute standard scheduling metrics:
    - Completion time.
    - Turnaround time (TAT).
    - Waiting time.
    - Average turnaround time and average waiting time.

#### Functions / Modules to Understand

- Scheduling algorithms provided by the practical:
  - FCFS.
  - SJF (preemptive).
  - NonSJF (non-preemptive SJF).
  - PriorityScheduling (preemptive priority).
  - NonPriorityScheduling (non-preemptive priority).
  - Round Robin (RR, preemptive with time quantum).
- Shared concepts:
  - How to select the next process when multiple processes are available.
  - How to update remaining time or execution state for preemptive algorithms.
  - How to update completion times when a process finishes.

#### What Needs to Be Implemented

- Implement each scheduler according to its policy:
  - FCFS: execute processes in order of arrival.
  - SJF preemptive: at each time unit, select the arrived process with the smallest remaining burst time.
  - SJF non-preemptive: when you select a shortest job among arrived processes, run it to completion.
  - Priority preemptive: at each time unit, select the highest-priority (or smallest numeric priority) among arrived processes with remaining burst time.
  - Priority non-preemptive: select a process by priority among arrived processes and run to completion.
  - Round Robin:
    - Maintain a ready queue.
    - Use a configurable time quantum.
    - Preempt and re-queue processes that do not finish within the quantum.
- Implement metric calculations:
  - TAT = completion_time - arrival_time.
  - Waiting time = TAT - burst_time (or equivalent computed formula based on remaining execution tracking).
  - Average metrics over all processes.

---

### Practical B2: Memory Placement Strategy

#### Experimental Requirements

- Simulate dynamic memory allocation where:
  - You have a set of memory blocks (with sizes).
  - You have a set of processes (requests) (with sizes).
  - Each process is allocated to at most one block using a specific placement strategy.
- After allocation:
  - Reduce the chosen block’s remaining size.
  - If no suitable block exists, report “Not Allocated” for that process.
- Support the following placement strategies:
  - First Fit.
  - Best Fit.
  - Next Fit.
  - Worst Fit.

#### Functions / Modules to Understand

- Each strategy’s selection rule:
  - First Fit: first block in order that fits.
  - Best Fit: smallest block that fits.
  - Next Fit: continue from the last checked block index cyclically.
  - Worst Fit: largest block that fits.
- Allocation outputs:
  - For each process: the allocated block index (or “not allocated”).
  - Remaining block size after allocation.

#### What Needs to Be Implemented

- Implement each placement strategy:
  - Iterate processes in input order.
  - For each process, choose a candidate block index based on the strategy rule.
  - Apply allocation by subtracting process size from block size.
  - Record allocation results and remaining block sizes.
- Implement output reporting:
  - Print a table with process number, process size, block number, and remaining block size (or not allocated).

---

### Practical B3: Page Replacement

#### Experimental Requirements

- Simulate page replacement for a reference string:
  - Input is a sequence of page numbers.
  - Input also includes the frame capacity.
  - Maintain frame contents and determine whether each reference is a hit or a fault.
- For each replacement algorithm, compute:
  - Total page hits.
  - Total page faults.
  - Hit ratio and fault ratio (as percentage).

#### Functions / Modules to Understand

- Algorithms in the practical:
  - FIFO.
  - LRU.
  - Optimal (OPT) page replacement.
- Replacement-state management:
  - What data structure represents frame order (queue, recency list, or future reference scanning).
  - How to update state on hits and after evictions.

#### What Needs to Be Implemented

- Implement FIFO:
  - Maintain a queue pointer for the next eviction frame.
  - On fault with full capacity, evict the oldest loaded page.
- Implement LRU:
  - Track recency order of pages currently in frames.
  - On fault with full capacity, evict the least recently used page (oldest in recency ordering).
- Implement Optimal:
  - On fault with full capacity, evict the page whose next use is farthest in the future (or never used again).
- Implement the simulation loop:
  - For each referenced page:
    - Detect hit vs fault.
    - Update frame contents accordingly.
    - Update hit/fault counters.
  - Optionally record a frame table snapshot per step (if the practical expects tabular output).

---

## Distributed System Practical

### Practical DS-1: Socket Programming

#### Experimental Requirements

- Implement a TCP socket-based client-server communication using Python.
- Server behavior:
  - Create a TCP server socket.
  - Bind to a host and port.
  - Listen for incoming connections.
  - Accept connections and handle each client concurrently (using threads in the provided structure).
- Client behavior:
  - Connect to the server.
  - Send messages entered by the user.
  - Receive and display responses from the server.

#### Functions / Modules to Understand

- `server.py`:
  - Socket setup (create socket, bind, listen, accept).
  - Connection handler loop (`handle_client`) that receives bytes and sends them back (echo).
  - Thread creation for each client.
- `client.py`:
  - Connection establishment and message send/receive loop.

#### What Needs to Be Implemented

- Implement server socket operations and concurrency:
  - Accept new connections in a loop.
  - Dispatch each accepted client socket to a handler running concurrently.
- Implement message handling:
  - Read incoming data from the client.
  - Process/forward it as required (in the provided structure it echoes back).
  - Close sockets when the connection ends.
- Implement client interaction:
  - Connect to server.
  - Send user-input messages and print responses until the user exits.

---

### Practical DS-2: RPC via XML-RPC

#### Experimental Requirements

- Implement an XML-RPC server and a client in Python.
- Server:
  - Provide a set of methods (e.g., arithmetic operations).
  - Restrict allowed RPC request paths for safety.
- Client:
  - Create a proxy to the server endpoint.
  - Call remote methods and print the returned results.

#### Functions / Modules to Understand

- `rpc_server.py`:
  - Define a class with callable RPC methods.
  - Register an instance with `SimpleXMLRPCServer`.
  - Restrict request handling paths (e.g., only `/RPC2`).
- `rpc_client.py`:
  - Create a `ServerProxy` to the server.
  - Call remote methods like `add(...)` and `subtract(...)`.

#### What Needs to Be Implemented

- Implement server-side RPC method registration:
  - Implement the method signatures expected by the client.
  - Ensure server can serve requests and return values.
- Implement client-side RPC calls:
  - Build the correct proxy URL.
  - Call methods with correct argument types.
  - Handle/display results.

---

### Practical DS-3: Election Algorithms (Ring Election and Bully Election)

#### Experimental Requirements

- Simulate coordinator election among nodes using:
  - Ring election: nodes pass an election message around the ring; the highest node ID becomes coordinator.
  - Bully election: a node that detects it might be coordinator contacts higher-ID nodes; if no higher node responds, it declares itself coordinator.
- Each node must track:
  - Node ID.
  - Current known coordinator.

#### Functions / Modules to Understand

- `ring_and_bully.py`:
  - Node initiation and state transitions.
  - Ring message forwarding among nodes.
  - Bully behavior sending messages to higher-ID nodes.

#### What Needs to Be Implemented

- Implement ring election logic:
  - Determine if current node has the maximum ID among participating nodes.
  - Forward election messages to the next node in the ring until termination.
  - Set coordinator when the highest ID node is determined.
- Implement bully election logic:
  - Identify higher-ID nodes.
  - If no higher-ID nodes exist, declare coordinator locally.
  - Otherwise send election messages to higher nodes and handle “no response” decision logic.
- Implement a simple simulation driver:
  - Create a list of nodes.
  - Trigger both election algorithms for demonstration.

---

### Practical DS-4: Clock Synchronization (Lamport Clock and NTP)

#### Experimental Requirements

- Implement clock synchronization mechanisms in Python demos:
  - Lamport logical clock:
    - Maintain and update logical time.
    - Compare clock values between two logical clocks.
  - NTP-based synchronization:
    - Use an NTP client to request time from a configured server.
    - Convert and print the synchronized time.

#### Functions / Modules to Understand

- `lamport.py`:
  - `LamportClock` class with `tick()` and `get_time()`.
  - A driver that simulates events and compares two clocks.
- `ntp.py`:
  - `sync_ntp_time(server)` using `ntplib`.
  - Main entry to select an NTP server and print synchronized local time.

#### What Needs to Be Implemented

- Implement Lamport logical clocks:
  - Initialize logical clock.
  - Provide increment/tick on events.
  - Provide comparison logic in the demo driver.
- Implement NTP synchronization:
  - Create and send a request to an NTP server.
  - Extract the response transmission time.
  - Convert to readable local time and print.

---

## DBMS Practical

### Practical DBMS-1: SQL DDL (Schema, Views, Indexes)

#### Experimental Requirements

- Create databases and define schemas using SQL DDL.
- Create tables with column definitions.
- Apply table alterations:
  - Add columns.
  - Define primary keys and foreign keys.
- Create derived objects:
  - Views.
  - Indexes (including composite indexes).
- Perform basic data definition operations:
  - Create table from query (`CREATE TABLE ... AS SELECT`).
  - Truncate/drop temporary or derived tables and views as required.

#### Functions / Modules to Understand

- DDL objects and operations:
  - `CREATE DATABASE`, `USE DATABASE`.
  - `CREATE TABLE`.
  - `ALTER TABLE ... ADD ...`.
  - `ALTER TABLE ... ADD PRIMARY KEY` and `FOREIGN KEY` constraints.
  - `CREATE VIEW` and `DROP VIEW`.
  - `CREATE INDEX` and basic index inspection.
  - `TRUNCATE`, `DROP TABLE`.

#### What Needs to Be Implemented

- Implement the full schema workflow for the assignment:
  - Create the main database(s).
  - Create base tables and apply constraints.
  - Insert sample data to demonstrate later steps (select queries).
  - Create a view and index it as requested.
  - Run the specified `ALTER` and cleanup operations (drop/truncate).

---

### Practical DBMS-2: SQL DML (Data Manipulation, Subqueries, Aggregates)

#### Experimental Requirements

- Implement SQL DML workflows:
  - Insert records into tables.
  - Update rows based on conditions.
  - Delete records based on conditions.
- Include selection features:
  - Conditional selection using `BETWEEN`.
  - Filtering with aggregate comparisons using subqueries (e.g., compare a column to `MAX(...)`).
  - Compute aggregates (e.g., `SUM`).
  - Union of result sets (`UNION`).

#### Functions / Modules to Understand

- DML operations:
  - `INSERT`.
  - `UPDATE`.
  - `DELETE`.
  - `SELECT` with:
    - `WHERE` conditions.
    - `BETWEEN`.
    - subquery filters.
    - `UNION`.
  - `CREATE TABLE ... AS SELECT` for derived data.

#### What Needs to Be Implemented

- Implement the end-to-end DML sequence:
  - Populate tables.
  - Apply updates and deletes.
  - Execute selection queries that use conditions, aggregates, and subqueries.
  - Produce a derived table using `CREATE TABLE ... AS SELECT`.
  - Use union to combine results.

---

### Practical DBMS-3: SQL Joins & Views

#### Experimental Requirements

- Create schemas involving at least two related tables (e.g., `cust_tab` and `cust_info`).
- Implement join queries:
  - Inner join.
  - Left outer join.
  - Right outer join.
- Implement a join-based view and query the view.

#### Functions / Modules to Understand

- Join types:
  - `INNER JOIN ... ON ...`
  - `LEFT OUTER JOIN ... ON ...`
  - `RIGHT OUTER JOIN ... ON ...`
- View creation:
  - `CREATE VIEW view_name AS SELECT ...`

#### What Needs to Be Implemented

- Implement:
  - Table creation and constraint setup (primary key and foreign key).
  - Join queries to produce combined results.
  - View definition that uses join conditions.

---

### Practical DBMS-4: SQL Connectivity (JDBC CRUD + Search/Update/Delete)

#### Experimental Requirements

- Implement a JDBC program that connects to a MySQL database.
- Perform basic CRUD-like operations:
  - Insert a user-specified number of records.
  - Select and display records.
  - Search a record by key.
  - Update records by key.
  - Delete records by key.
- Use JDBC `Statement` and `PreparedStatement` patterns for parameterized operations.
- Print intermediate results after each major operation stage.

#### Functions / Modules to Understand

- JDBC workflow:
  - `DriverManager.getConnection(...)`.
  - `createStatement()` and `prepareStatement(...)`.
  - Execute updates (`executeUpdate`) and queries (`executeQuery`).
  - Iterate `ResultSet` and display fields.
  - Close the connection.

#### What Needs to Be Implemented

- Implement a complete connectivity demo:
  - Connection setup using placeholder database credentials.
  - Insert loop for N records using prepared statements.
  - Select-display, search-display, update-display, and delete-display steps.

---

### Practical DBMS-5: Basic PL/SQL (Stored Procedures for Business Logic)

#### Experimental Requirements

- Implement stored procedures to:
  - Calculate derived values based on table data and current date (business rule).
  - Update status fields and maintain related tables (e.g., inserting into a “fine” table and removing entries on submission).
- Provide a procedure call sequence that demonstrates both calculation and update stages.

#### Functions / Modules to Understand

- PL/SQL stored procedure features:
  - Procedure parameters (input values).
  - Local variable declarations.
  - `SELECT ... INTO` to fetch data into variables.
  - Conditional logic with `IF/ELSE` or multiple `IF` blocks.
  - Procedure side-effects on tables.

#### What Needs to Be Implemented

- Implement:
  - `calc_Fine`-like procedure that computes a numeric result from date difference and conditional thresholds.
  - `submit`-like procedure that updates a status field and cleans related derived records.
  - Provide calls that run the procedures and then query outputs to verify changes.

---

### Practical DBMS-6: PL/SQL Trigger (Audit on UPDATE)

#### Experimental Requirements

- Implement a trigger that fires `BEFORE UPDATE` on a base table.
- For each updated row, insert audit information into an audit table:
  - Store old vs new values as required by the schema.
- Perform sample updates that demonstrate the trigger behavior.

#### Functions / Modules to Understand

- Trigger definitions:
  - `CREATE TRIGGER ... BEFORE UPDATE ... FOR EACH ROW`.
  - Use of `OLD` and `NEW` row contexts to access old and new values.
  - Insert into audit table within the trigger body.

#### What Needs to Be Implemented

- Implement:
  - Base table update logic.
  - Audit table structure.
  - Trigger body that inserts `(row key, old value, new value)` into the audit table.

---

### Practical DBMS-7: PL/SQL Cursor (Cursor-Based Data Transfer)

#### Experimental Requirements

- Implement procedures that use cursors to:
  - Iterate over rows in one table and synchronize or merge records into another table.
  - Use an exit handler or `NOT FOUND` logic to terminate cursor iteration.
- Provide at least:
  - A procedure that loops over two cursors and inserts missing records.
  - A procedure that takes a parameter and copies rows meeting a filter condition, checking for existence in the destination table.

#### Functions / Modules to Understand

- Cursor and handler concepts:
  - Declaring cursors with `SELECT` queries.
  - Opening and fetching cursor rows.
  - Detecting end-of-data using `CONTINUE HANDLER` / `EXIT HANDLER`.
  - `IF` existence checks (`NOT EXISTS` / subquery checks).

#### What Needs to Be Implemented

- Implement cursor-based procedures:
  - Procedure P1 without parameters: synchronize old/new tables by comparing keys.
  - Procedure P2 with an input threshold: copy only records beyond a given key and avoid duplicates.
  - Include calls that demonstrate both procedures.

---

### Practical DBMS-8: PL/SQL Stored Procedure & Function (Parameterized Grade/Classification)

#### Experimental Requirements

- Implement:
  - A stored procedure with input and output parameters to compute a classification (e.g., grade).
  - A deterministic function that returns the classification using the stored procedure.
- Use table queries to compute a numeric value, then map it into named categories.
- Update a target table with the computed classification.

#### Functions / Modules to Understand

- Procedure features:
  - IN/OUT parameters.
  - Conditional mapping using `IF/ELSEIF/ELSE`.
  - Updating table rows within the procedure.
- Function features:
  - `RETURNS` type.
  - Calling the procedure to fill the OUT parameter and returning it.
  - Deterministic function attribute if required by the SQL dialect.

#### What Needs to Be Implemented

- Implement:
  - `proc_Grade`-like procedure with IN roll/key and OUT grade/class string.
  - `func_Grade`-like function that wraps the procedure and returns the grade.
  - Update logic that writes the grade to a class/category column.

---

### Practical DBMS-9: MongoDB CRUD

#### Experimental Requirements

- Implement MongoDB connection and CRUD operations for a collection:
  - Insert multiple documents with user input fields.
  - Query and print documents.
  - Delete documents by selection criteria.
  - Optionally drop the database and demonstrate restart/loop behavior.

#### Functions / Modules to Understand

- MongoDB Java driver workflow:
  - Connect to local MongoDB instance.
  - Access database and create collection.
  - Insert documents (e.g., with `BasicDBObject`).
  - Query documents via `find()` and print selected fields.
  - Delete documents via `remove()`/`delete` patterns.

#### What Needs to Be Implemented

- Implement:
  - Collection initialization.
  - Insert loop for N documents.
  - Display of inserted data.
  - Delete loop based on user input.
  - Optional database drop and repeat/continue logic.

---

### Practical DBMS-10: MongoDB Connectivity (Java Driver CRUD Interaction)

#### Experimental Requirements

- Implement a Java-based MongoDB connectivity demo that supports:
  - Connection confirmation.
  - Insert records through a loop.
  - Delete records through a loop.
  - Query and print results between operations.

#### Functions / Modules to Understand

- Same MongoDB driver patterns:
  - Mongo client creation.
  - DB/collection selection.
  - CRUD via driver APIs.

#### What Needs to Be Implemented

- Implement the CRUD interaction flow:
  - Insert N records.
  - Display after insert.
  - Delete M records.
  - Display after delete (and optionally drop database).

---

### Practical DBMS-11: MongoDB Indexing & Aggregation

#### Experimental Requirements

- Implement aggregation queries over a MongoDB collection:
  - Group by a field (e.g., customer name).
  - Compute multiple aggregation metrics:
    - `sum`, `avg`, `min`, `max`, and other aggregations supported by the practical.
    - Demonstrate group-accumulator variety (e.g., `first`, `last`, `addToSet`).
- Implement index operations:
  - Create indexes on selected fields.
  - Inspect indexes.
  - Drop indexes.

#### Functions / Modules to Understand

- Aggregation pipeline basics:
  - `$group` operator.
  - Accumulator functions.
- Index management:
  - `createIndex`, `getIndexes`, `dropIndex`.

#### What Needs to Be Implemented

- Implement:
  - A set of aggregation queries with different accumulators for the same grouping key.
  - Index creation and deletion steps with verification of index lists.

---

### Practical DBMS-12: MongoDB MapReduce

#### Experimental Requirements

- Implement MongoDB `mapReduce`:
  - Define a map function that emits key/value pairs from documents.
  - Define a reduce function that aggregates emitted values per key.
  - Apply a filter query (e.g., based on a status field) before running map-reduce.
  - Store output into a named collection and query it afterward.

#### Functions / Modules to Understand

- `mapReduce` workflow:
  - Map function: `emit(key, value)`.
  - Reduce function: aggregate `values` for each key.
  - Output options (e.g., output collection name).

#### What Needs to Be Implemented

- Implement:
  - Map and reduce functions.
  - A query filter used for mapReduce input set.
  - Execute mapReduce and query output collections.

---

### Practical DBMS-13: MongoDB Query Operators (Complex Conditions)

#### Experimental Requirements

- Implement a MongoDB query set demonstrating various logical and comparison operators:
  - `$or`, `$and`-style combinations.
  - `$ne` and `$nor`.
  - `$not` with nested comparison operators (e.g., not greater than).
- Combine these operators to fetch documents based on specified criteria.

#### Functions / Modules to Understand

- MongoDB query operator semantics:
  - Logical operators for combining conditions.
  - Comparison operators for matching document field values.

#### What Needs to Be Implemented

- Implement:
  - Several queries using distinct operator combinations.
  - Demonstrate correctness by printing returned documents.

