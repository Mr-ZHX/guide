# HIT-DatabaseSystem (Labs in `lab/`) — English Lab Manual

This document provides an English write-up for the experiments implemented under:
`HIT-DatabaseSystem/lab/`

The `lab/` directory contains three code modules:

1. `Flight/` — a Flight Management system (Qt GUI + MySQL)
2. `QueryProcessing/` — external-memory relational operators and join algorithms (C/C++ style simulation)
3. `QueryOptimize/` — query parsing into an operator tree and a simple heuristic optimizer

---

## 0. Project overview

According to `HIT-DatabaseSystem/README.md`, the coursework includes:

- **Lab 1**: Database system development (a flight management system with CRUD + booking)
- **Lab 2**: Query processing and query optimization (selection/projection/join + a simple optimization)

In the current repository, Lab 1 corresponds to `lab/Flight`, and Lab 2 corresponds to `lab/QueryProcessing` and `lab/QueryOptimize`.

---

## Lab 1 — Flight Management System (Qt + MySQL)

### 1. Objectives

- Build a GUI application that interacts with a relational database.
- Perform common database operations (query, insert, delete) through SQL statements.
- Practice linking a GUI workflow (multiple windows/pages) to database queries.

### 2. Architecture (from code)

**Main entry**

- `lab/Flight/main.cpp` starts the Qt application and shows the `Parent` window.

**Window/page controller**

- `Parent` (`parent.h/.cpp`) provides navigation buttons:
  - Airport page
  - Flight page
  - Booking page
  - Passenger page

**Database access**

- `sqlConnect(QSqlDatabase& sql)` in `sql.cpp` creates/connects a MySQL database connection via Qt SQL.

Key connection settings in `sqlConnect`:

- Host: `127.0.0.1`
- User: `root`
- Password: `your_password` (placeholder in code)
- Port: `3306`
- Database name: `flight`

### 3. Functional modules and SQL statements

The implemented features are driven by the SQL queries in each widget class.

#### 3.1 Airport queries (`airport.cpp`)

- `queryDeparture()`
  - Reads `airport_id` from a line edit and runs:
    - `select * from airport_dep where airport_id = :qId order by dep_time asc`
- `queryArrival()`
  - Runs:
    - `select * from airport_arv where airport_id = :qId order by arv_time asc`

The UI table headers suggest columns like:
`airport_id`, airport name, `flight_id`, plane model, airline name, and departure/arrival time.

#### 3.2 Flight queries (`flight.cpp`)

- `queryDepArv()`
  - Uses `flight.id` as the filter and runs an `EXISTS` subquery:
    - `select departure, dep_city, arrival, arv_city, dpt_time, arv_time from flight_arv_dep where exists( select * from flight where flight.id = :qId and flight_arv_dep.flight_id = flight.id)`
- `queryPassenger()`
  - Runs:
    - `select passenger_id, name, gender, age, seat, luggage_count from flight_passenger where exists(select * from flight where flight.id = :qId and flight_passenger.flight_id = flight.id) order by passenger_id`
- `queryStaff()`
  - Runs:
    - `select staff_id, name, position from flight_staff where exists(select * from flight where flight.id = :qId and flight_staff.flight_id = flight.id order by staff_id)`

#### 3.3 Passenger management (`Passenger.cpp`)

- `queryPassenger()`
  - `select * from passenger where id = :qId`
- `addPassenger()`
  - Inserts into `passenger` with optional fields:
    - If both gender and age are present: `INSERT INTO passenger (id, name, gender, age) ...`
    - Otherwise inserts with fewer columns.
- `deletePassenger()`
  - Deletes by id:
    - `DELETE FROM passenger WHERE id = :id`

#### 3.4 Booking and ticket insertion (`booking.cpp`)

- `queryFlight()`
  - Reads:
    - departure city and arrival city from UI
    - optionally an arrival time constraint (`QTime`)
  - If time is provided:
    - queries `booking_dep_arv` with:
      - `... group by flight_id having arv_time < :arv_time order by arv_time asc`
  - Otherwise:
    - queries `booking_dep_arv ... order by arv_time asc`
- `addBooking()`
  - Inserts:
    1. `booking(passenger_id, flight_id, state, price)`
    2. `ticket(passenger_id, flight_id, seat)`

The code uses fixed defaults:

- `state = "success"`
- `price = "0"`
- `seat = "Test Seat"`

### 4. Build and run

The `Flight/CMakeLists.txt` is configured for Windows (Qt 5.14.2 MinGW) and links MySQL:

- `CMAKE_PREFIX_PATH` is hardcoded to the Qt installation path
- MySQL include/lib directories are hardcoded (`D:/Code/MySQL8.0/...`)

**Build**

- Build `lab/Flight` with CMake (ensure your local Qt/MySQL paths match the CMake file).

**Run**

1. Start MySQL and make sure the `flight` database exists with the tables used in the SQL statements (e.g., `airport_dep`, `airport_arv`, `flight`, `passenger`, `booking`, `ticket`, etc.).
2. Update `your_password` in `sql.cpp`.
3. Run the compiled `Flight` executable.

### 5. Verification (what to check)

- Navigation: clicking each button correctly opens the corresponding page.
- Database connection: queries show results or display `sql.lastError().text()` if connection fails.
- Correctness of queries:
  - Airport page shows ordered departure/arrival flights for the given `airport_id`.
  - Flight page returns matching departure/arrival routes and passengers/staff for a flight id.
- CRUD operations:
  - Passenger add/delete works and table refresh reflects the DB state.
- Booking:
  - Booking insert succeeds and `ticket` is created with the inserted `passenger_id` and `flight_id`.

---

## Lab 2 — Query Processing (External Memory Operators)

### 1. Objectives

Implement relational operators under an **external-memory / block-based** model:

- Selection (`select`)
- Projection (`project`)
- Joins:
  - Nested-loop join (`nlj`)
  - Hash join (`hj`)
  - Sort-merge join (`smj`)

All algorithms run on simulated block storage:

- Data blocks are stored as files named `<address>.blk`.
- A fixed-size buffer controls in-memory pages (buffer overflow affects block allocation).

### 2. Key data model and constants (from `file.h`)

The code models two relations:

- `R(A, B)` as `Relation { attr1, attr2 }`
- `S(C, D)` as `Relation { attr1, attr2 }`

Important constants:

- Buffer:
  - `BUF_SIZE = 520`
  - `BLK_SIZE = 64`
  - `PTR_SIZE = 4`
- Data layout:
  - `TUPLE_PER_BLK = 7`
  - `R_NUM = 112`, `S_NUM = 224`
  - Therefore `R_BLK_NUM = 16`, `S_BLK_NUM = 32`
- Join/output block addresses:
  - `SELECT_RET_ADDR = 1000`
  - `PROJECT_RET_ADDR = 2000`
  - `NLJ_RET_ADDR = 3000`
  - `HJ_RET_ADDR = 4000`
  - `SMJ_RET_ADDR = 5000`

### 3. External memory support code

#### 3.1 Buffer and block I/O (`extmem.*`)

- `initBuffer(...)` allocates the in-memory buffer and tracks:
  - number of blocks in buffer
  - number of free blocks
  - I/O count (`numIO`)
- `readBlockFromDisk(addr, buf)` opens file `${addr}.blk` in binary mode and reads:
  - up to `blkSize` bytes into the allocated buffer region
- `writeBlockToDisk(blkPtr, addr, buf)` writes `blkSize` bytes to `${addr}.blk`

Blocks are “bookkept” inside the buffer via markers:

- `BLOCK_AVAILABLE = 0`
- `BLOCK_UNAVAILABLE = 1`

#### 3.2 Block packing/unpacking (`blkio.*`)

- `readTuplesFromBlock(blk, Relation* R)`
  - copies `7` tuples, each tuple stored as 8 bytes:
    - `attr1` (int, 4 bytes)
    - `attr2` (int, 4 bytes)
- `output(...)`
  - prints results and writes them as a 3-int record (`Relation3 attr1/attr2/attr3`) using 12 bytes per output tuple.

### 4. Operator implementations

#### 4.1 Selection (`select.cpp`)

Implements:

- `select * from R where R.A = 40`  (prints and writes tuples where `attr1 == 40`)
- `select * from S where S.C = 60`  (prints and writes tuples where `S.attr1 == 60`)

It reads each input block, iterates tuples, filters, writes to the selection return blocks, and prints total `numIO`.

#### 4.2 Projection (`project.cpp`)

Implements:

- `project A from R` with deduplication.

Flow:

1. `mergeSort(buf, SORT_R)` sorts `R` by `attr1` (A).
2. Reads sorted result blocks and outputs only when `R[j].attr1 != lastA`.

This relies on the sorted order to remove duplicates.

#### 4.3 Nested-loop join (`nestedLoopJoin.cpp`)

Implements:

- `R join S` on condition `R.A = S.C` (coded as `R[l].attr1 == S[k].attr1`).

Block nested-loop style:

1. Uses `M-1` free buffer blocks to read a chunk of `R`.
2. For each `R` chunk, scans all blocks of `S`.
3. Nested loops over tuples in the blocks to output matches.

#### 4.4 Hash join (`hashJoin.cpp`)

Implements:

- `R join S` on `R.A = S.C`.

Flow:

1. Hash partition `R` and `S`:
   - bucket key is `attr1 % BUCKET_NUM` where `BUCKET_NUM = 6`
2. For each bucket key:
   - load all `R` tuples in memory
   - iterate corresponding `S` bucket blocks and probe matches

#### 4.5 Sort-merge join (`sortMergeJoin.cpp`)

Implements:

- `R join S` on `R.A = S.C`.

Flow:

1. Sort both `R` and `S` using external merge sort:
   - `mergeSort(buf, SORT_R)` and `mergeSort(buf, SORT_S)`
2. Merge join by scanning sorted sequences and advancing the side with the smaller current key.
3. Handles duplicates by emitting all combinations for the same key value.

#### 4.6 External merge sort (`mergeSort.cpp`)

Implements a two-pass multi-way external merge sort:

- **Run creation (step 1)**:
  - split input blocks into segments
  - sort each segment in memory
  - write sorted runs to intermediate merge segment files
- **Merge (step 2)**:
  - repeatedly select the current minimum tuple among all segments
  - output to the final sorted return blocks

### 5. Data generation and running

The code includes:

- `genData.py` to generate `./data/*.blk`

Run-time requirement:

- `extmem.cpp` reads input blocks as `${addr}.blk` from the current working directory.
- Therefore, run `QueryProcessing` with the working directory set to:
  - `HIT-DatabaseSystem/lab/QueryProcessing/data/`

Example workflow:

1. Compile with CMake in `lab/QueryProcessing/`.
2. Ensure `data/` contains block files `0.blk ...` for input relations.
3. Run the produced `QueryProcessing` binary with CWD = `lab/QueryProcessing/data/`.
4. Use commands in the interactive prompt:
   - `select`
   - `project`
   - `nlj`
   - `hj`
   - `smj`
   - `exit`

### 6. Verification (what to check)

- The program prints `numIO` / `total numIO` for each operation.
- Output tuples are printed during selection and join operations.
- Output blocks are created as files at the configured return addresses (e.g., `1000.blk`, `3000.blk`, ... depending on command).

For a stronger verification, compare expected join/select/project results against the known generated dataset (or run with different generated seeds and observe stable logical behavior).

---

## Query Optimization (`lab/QueryOptimize`)

### 1. Objectives

- Parse a simplified SQL-like query string into an operator tree.
- Apply a heuristic rewrite to improve the plan structure.

### 2. Implementation overview

Key file:

- `lab/QueryOptimize/main.cpp`

Main components:

1. `TreeNode` (`tree.h`)
   - types: `SELECT`, `PROJECTION`, `JOIN`, `LEAF`
   - each node stores `info` for selection condition text
2. `parseQuery(...)`
   - tokenizes the query by spaces
   - builds a tree using parentheses and keywords
   - stores conditions inside `[ ... ]` into `TreeNode.info`
3. `optimizeQuery(...)`
   - **For SELECT**:
     - splits conditions by `&`
     - optimizes recursively
   - **For JOIN**:
     - creates `SELECT` nodes above the join children (predicate push-up/push-down style in tree form)
   - **For PROJECTION**:
     - recursively optimizes its child

The code prints the operator tree using pre-order traversal:

- `root->left->preOrder(0)` before and after optimization.

### 3. Example queries (from code)

`main.cpp` contains three test queries:

- `SELECT [ ENAME = 'Mary' & DNAME = 'Research' ] ( EMPLOYEE JOIN DEPARTMENT )`
- `PROJECTION [ BDATE ] ( SELECT [ ENAME = 'John' & DNAME = 'Research' ] ( EMPLOYEE JOIN DEPARTMENT ) )`
- `SELECT [ ESSN = '01' ] ( PROJECTION [ ESSN, PNAME ] ( WORKS_ON JOIN PROJECT ) )`

### 4. How to run and verify

Build and run the `QueryOptimize` executable, and verify:

- The printed pre-order tree shows:
  - the initial parsed plan
  - and the optimized plan where predicates are inserted/rewritten around join/projection/selection nodes.

---

## Suggested submission content (English write-up)

For a “complete” lab submission, you can include:

- For `Flight/`:
  - screenshots of each page (airport/flight/passenger/booking)
  - a short description of key SQL queries used by each page
  - an explanation of the database connection settings (`sql.cpp`)
- For `QueryProcessing/`:
  - explanation of the external-memory block model (buffer, block files)
  - description of each operator implementation and its algorithmic strategy
  - representative outputs and `numIO` numbers from interactive commands
- For `QueryOptimize/`:
  - the before/after operator tree outputs for the sample queries

