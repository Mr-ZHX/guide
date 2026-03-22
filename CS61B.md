# skeleton-sp24 — Labs & Projects Overview (English)

This document summarizes **what each numbered folder in this repository is for**, using **only the skeleton layout, filenames, and public comments** (no implementation steps). It is meant for readers who have **not** taken the course. It does **not** paste student write-ups or external lab reports.

**Context (inferred from naming and authors in source headers):** The tree matches the **UC Berkeley CS61B (Spring 2024–style)** public **skeleton**: weekly **`lab01`–`lab10`**, homework **`hw0b`** and **`hw2`**, and larger **`proj0`–`proj3`** milestones. Official grading rules and deadlines belong to the **CS61B** materials for your semester.

---

## Homework (not labeled `lab`)

### `hw0b`

**Contents:** `JavaExercises`, `ListExercises`, `MapExercises` with matching JUnit tests (`JavaExercisesTest`, `ListExercisesTest`, `MapExercisesTest`), plus `DessertTest`.

**Purpose (inferred):** early **Java** practice—basic syntax, **lists**, and **maps**—before heavier data structures.

---

### `hw2`

**Contents:** `Percolation`, `PercolationStats`, visualization helpers (`PercolationPicture`, `InteractivePercolationVisualizer`), and many **input grid files** under `inputFiles/`.

**Purpose (inferred):** the classic **percolation** Monte Carlo assignment—model an **N×N** grid that opens sites and test whether the system **percolates**; estimate **threshold** statistics; optional **visualization**.

---

## Weekly labs (`lab01`–`lab10`)

### `lab01`

**Contents:** `Arithmetic` with a deliberately wrong `sum` (uses multiplication), `ArithmeticTest`.

**Purpose:** introduce **JUnit**, **debugging**, and fixing logic so tests pass (simple arithmetic utilities).

---

### `lab02`

**Contents:** `bomb` package (`Bomb`, `BombMain`), `common/IntList`, `BombTest`.

**Purpose:** **debugger** practice—navigate multi-phase “bomb” style code, inspect `IntList` structures, and satisfy hidden phase checks (without changing locked starter files where noted).

---

### `lab03`

**Contents:** `adventure` package with staged tasks (`BeeCountingStage`, `MachineStage`, `PalindromeStage`, `SpeciesListStage`, `FillerStage`, `AdventureGame`), `puzzle/Puzzle`, shared `IntList`, rich **test data** under `tests/data/`.

**Purpose:** string/list **manipulation** and small **game** flows; optional **puzzle** component; reinforces incremental testing across **stages**.

---

### `lab04`

**Contents:** *This directory is **empty** in this repository snapshot (no source files).*

**Purpose:** unknown from local files—if your course uses Lab 4, obtain the skeleton from **course infrastructure** (website / Gradescope / git template).

---

### `lab05`

**Contents:** `UnionFind` with TODO methods, `UnionFindTest`.

**Purpose:** implement a **disjoint-set (union–find)** structure with **path compression** and **union-by-size** (behavior specified in method comments and tests).

---

### `lab06`

**Contents:** `Map61B` interface, reference `ULLMap`, tests for **`BSTMap`** (your class is not pre-supplied—only the interface + tests), speed tests.

**Purpose:** implement a **binary search tree map** (`BSTMap`) satisfying `Map61B` operations (put/get/clear/etc.) and pass correctness + basic **performance** checks.

---

### `lab07`

**Contents:** `RedBlackTree` stub, `TestRedBlackTree`.

**Purpose:** implement or complete a **left-leaning red–black** (or course-specified) **balanced BST** variant per tests.

---

### `lab08`

**Contents:** `hashmap` package with `Map61B`, `ULLMap`, **`MyHashMap`** stub, extensive tests (`TestMyHashMap`, buckets/extra tests) and **insert speed** tests.

**Purpose:** implement a **hash table**-based `Map61B` with **separate chaining** (bucket lists) and validate behavior and asymptotic performance on prescribed workloads.

---

### `lab09`

**Contents:** `tileengine` (`TETile`, `TERenderer`, `Tileset`), `utils/FileUtils`, `gameoflife/GameOfLife` with demos, **Game of Life** pattern files (`patterns/*.txt`), `GameOfLifeTests`, save/load test files.

**Purpose:** **Conway’s Game of Life** using the course **tile engine** and **StdDraw**-style rendering; evolve a grid over time; support **saving/loading** world state via files (behavior covered by tests).

---

### `lab10`

**Contents:** same **tile engine** utilities as Lab 9, plus `tetris` (`Tetris`, `Tetromino`, `Movement`, `BagRandomizer`).

**Purpose:** build a **Tetris**-style game on the tile grid—**piece** shapes, **movement**, randomization—using the shared rendering stack (no full solution code in the skeleton).

---

## Projects (`proj0`–`proj3`)

### `proj0` — 2048 (game logic)

**Contents:** `game2048logic/Model` with many **TODO** tasks; rendering package `game2048rendering` (`Board`, `GUI`, `Game`, …); extensive logic tests (`TestTiltMerge`, `TestNbyN`, `TestModel`, …).

**Purpose:** implement **2048** **board logic**: empty cells, max tile detection, move existence, **tilt** mechanics (merge rules, score updates) for arbitrary board sizes and sides—**UI** is largely provided.

---

### `proj1a` — Deques (linked list)

**Contents:** `Deque61B` interface, students implement **`LinkedListDeque61B`**, tests (`LinkedListDeque61BTest`, `PreconditionTest`).

**Purpose:** doubly linked **deque** with required **API** (add/remove ends, iteration, `get`, `equals`, … per spec/tests).

---

### `proj1b` — Deques (array-based)

**Contents:** same `Deque61B` contract, implement **`ArrayDeque61B`**, circular-buffer style tests.

**Purpose:** **resizing array** deque with same external behavior as Proj 1A, with **efficiency** expectations.

---

### `proj1c` — Data structures + audio

**Contents:** both deque implementations, **`GuitarString`** (Karplus–Strong **string** simulation using a deque as a ring buffer), `GuitarPlayer`, `GuitarHeroLite`, tests (`TestGuitarString`, `MaxArrayDeque61BTest`).

**Purpose:** reuse **Proj 1B** deque to synthesize **audio** via Karplus–Strong; connect CS fundamentals to a small **interactive** sound demo.

---

### `proj2a` — NGrams (data & web)

**Contents:** `ngrams/TimeSeries`, `NGramMap` (TODO), **Spark** HTTP server (`NgordnetServer`, `NgordnetQueryHandler`, `NgordnetQuery`), dummy history handlers, plotting demos, tests for time series and history handlers.

**Purpose:** load **Google NGrams**-style datasets; implement **year → count** `TimeSeries` operations; answer **word-count history** queries; serve results through a **local web API** for visualization.

---

### `proj2b` — WordNet hyponyms (part B)

**Contents:** empty **`HyponymsHandler`**, Spark server wiring like 2A, tests for **hyponym** queries (`TestOneWordK0Hyponyms`, `TestMultiWordK0Hyponyms`).

**Purpose:** given **WordNet**-like graph data, return **hyponyms** of words for the browser UI (**k = 0** cases in test names).

---

### `proj2c` — WordNet hyponyms (part C)

**Contents:** same browser stack; additional tests (`TestOneWordKNot0Hyponyms`, `TestCommonAncestors`).

**Purpose:** extend Proj 2B with **non-zero k** (popularity / ranking style constraints as defined by course data) and **common-ancestor** style queries on the semantic graph.

---

### `proj3` — BYOW (Build Your Own World)

**Contents:** `tileengine` (`TETile`, `TERenderer`, `Tileset`), `utils` (`RandomUtils`, `FileUtils`), empty **`World`**, stub **`Main`**, `AutograderBuddy`, `WorldGenTests`.

**Purpose:** **procedural world generation** and exploration game: design **hallways/rooms** (or course-specific layout rules), **pseudo-random** worlds with **seed**, **save/load**, and **keyboard** interaction using the tile engine—**core gameplay** is student-authored inside `World` / `Main`.

---

## Quick reference

| Folder | Theme |
|--------|--------|
| `hw0b` | Basic Java, lists, maps |
| `hw2` | Percolation |
| `lab01` | Tests + debugging (`Arithmetic`) |
| `lab02` | Debugger + `IntList` / bomb |
| `lab03` | Adventure stages + puzzle |
| `lab04` | *Empty in this clone* |
| `lab05` | `UnionFind` |
| `lab06` | `BSTMap` |
| `lab07` | `RedBlackTree` |
| `lab08` | `MyHashMap` |
| `lab09` | Game of Life + tiles |
| `lab10` | Tetris + tiles |
| `proj0` | 2048 logic |
| `proj1a–c` | Deques → guitar string |
| `proj2a–c` | NGrams → WordNet hyponyms + web |
| `proj3` | BYOW |

---

## Disclaimer

- This repository is a **skeleton**: many methods are **`TODO`** or empty; **tests** encode expected behavior.
- **Lab 4** is not recoverable from this tree alone.
- Always use the **official CS61B website** / **spec PDFs** for exact **deliverables**, **style**, and **submission** rules.
