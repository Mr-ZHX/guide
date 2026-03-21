# CS61B-Style Course Skeleton (`skeleton-sp25`) — English Lab Overview

This repository snapshot contains numbered Java projects (`proj0`, `proj0_hardmode`, `proj1a`, `proj1b`, `proj2a`, `proj2b`, `proj3`). The layout matches the **UC Berkeley CS61B** (Data Structures) project sequence for a typical semester (here labeled **sp25** in the folder name).

This document explains **what each lab is for**, **what you are expected to build**, and **what “done” means at a high level**. It does **not** describe implementation recipes and does **not** copy anyone’s lab report.

---

## Lab `proj0` — 2048 Game Logic (incremental)

### What you do
Implement the **core mechanics** of a 2048-style sliding puzzle on a 2D integer board, without focusing on the graphical front end first. Work is staged in small methods inside `game2048logic.GameLogic`, with automated tests per task.

### What functionality you implement
1. **Moving and merging tiles upward** in a column under constraints (e.g., how far a tile can slide, when two tiles merge).
2. **Tilting a single column upward** (stacking tiles, applying merge rules).
3. **Tilting the whole board “north”** (all columns).
4. **Tilting the board in all four directions** by reusing the north-tilt behavior together with board rotations (the skeleton references helpers for rotating the matrix).

### Success looks like
All unit/integration tests under `proj0/tests/` pass: the board transformations match the reference behavior for each task.

---

## Lab `proj0_hardmode` — 2048 “Hard Mode” (single deliverable)

### What you do
A variant where you implement **full-board tilting in all directions** in one place (`GameLogic.tilt`), typically by composing rotations and a core “tilt north” routine. The starter comments warn against cramming everything into one giant method—conceptually you still need correct behavior for **north, east, south, west**.

### What functionality you implement
The same **directional tilt semantics** as in `proj0`, but organized as one consolidated assignment rather than many staged TODOs.

### Success looks like
Tests (and/or manual runs with the provided GUI package) show that every tilt direction updates the board consistently with 2048 rules.

---

## Lab `proj1a` — Deques I (`LinkedListDeque61B`)

### What you do
Implement the `Deque61B<T>` abstract data type using a **linked-list-backed deque** (name implied by `LinkedListDeque61BTest`).

### What functionality you implement
A deque that supports (per `Deque61B.java`): add/remove at **front and back**, size/emptiness, iteration or listing (`toList`), indexed `get` (with performance expectations per course spec), and correct handling of edge cases (tests include precondition checks).

### Success looks like
`LinkedListDeque61BTest` and related tests pass; the implementation respects the interface contracts and any stated asymptotic expectations from the course materials.

---

## Lab `proj1b` — Deques II, Generics Utilities, and Audio (`ArrayDeque61B` + Karplus–Strong)

### What you do
1. Implement **`ArrayDeque61B`** — the same `Deque61B` API as in 1A, backed by a **resizing circular array** (tests: `ArrayDeque61BTest`).
2. Implement **`Maximizer61B`** — a small generic utility for finding maxima under constraints (`Maximizer61BTest`).
3. Implement **`GuitarString`** — a **Karplus–Strong** string simulation using your deque as a ring buffer (`TestGuitarString`).
4. Run the provided **Guitar Hero Lite**-style demo (`GuitarHeroLite`, `GuitarPlayer`, etc.) once audio works.

### What functionality you implement
- **Deque**: fast front/back operations without shifting the whole array; dynamic resizing.
- **Guitar string**: initialize a buffer sized from frequency, **pluck** with noise, **tic** to advance the physical simulation, **sample** the current wave value.
- **Optional course flavor**: `TTFAF` suggests an extended demo piece.

### Success looks like
Unit tests pass; the string simulation produces audible output when wired to the course audio utilities (as defined by the skeleton).

---

## Lab `proj2a` — NGrams I (`TimeSeries` + `NGramMap` + web browser)

### What you do
Build data structures for analyzing **historical word frequency** from Google **Ngrams**-style CSV inputs, and connect them to a small **web server** that plots word histories.

### What functionality you implement
1. **`TimeSeries`**: map **year → numeric value** with analysis helpers (copying ranges, summing/averaging weighted series, division, etc.—methods appear as TODOs in the skeleton).
2. **`NGramMap`**: load **word count history** and **total corpus counts per year** from files; answer queries like per-word history, total counts, and combined analyses across words.
3. **HTTP handlers**: serve plots via the provided Spark-based browser classes (`NgordnetServer`, `NgordnetQueryHandler`, etc.).

### Success looks like
Unit tests (`TimeSeriesTest`, `NGramMapTest`, `HistoryTextHandlerTest`, …) pass, and the local web UI can display plots for valid queries.

---

## Lab `proj2b` — WordNet Hyponyms + NGrams (`HyponymsHandler`)

### What you do
Extend the Ngrams browser so users can ask for **hyponyms** (more specific meanings) of words using a **WordNet**-like graph from synset/hyponym files, optionally **ranked by corpus frequency** over a year range.

### What functionality you implement
1. **Graph / ontology queries**: given one or more input words, compute the set (or top-**k** list) of hyponyms according to the WordNet data format used by the course.
2. **Integration with NGrams**: when `k > 0`, use frequency data to **order** hyponyms meaningfully (the autograder compares exact string outputs for fixed datasets).
3. **`HyponymsHandler`**: implement the handler wired through `AutograderBuddy.getHyponymsHandler(...)` so JSON responses match expected lists.

### Success looks like
Tests such as `TestOneWordK0Hyponyms`, `TestMultiWordK0Hyponyms`, etc., pass—your handler returns the exact canonical string for benchmark queries on the provided small/large WordNet slices.

---

## Lab `proj3` — BYOW: Build Your Own World (2D tile world)

### What you do
Create a **2D explorable world** using the provided **tile engine** (`TETile`, `TERenderer`, `Tileset`) and your own world model (`core.World`, entry in `core.Main`).

### What functionality you implement (typical CS61B BYOW scope)
1. **Procedural world generation**: pseudorandom but deterministic enough to support reproducibility requirements common in the spec (e.g., seeds).
2. **Rendering**: represent the world as a grid of `TETile`s and draw it with `TERenderer`.
3. **Interactivity** (as required by the course version you follow): often includes **keyboard-driven movement**, **HUD**, and sometimes **save/load** of world state—exact features are defined by the official spec, not duplicated here.

### Success looks like
Running `Main` launches your game/world; course autograders (if used) check required behaviors for world layout, gameplay, and any stated persistence or input rules.

---

## Summary Table

| Folder | Theme | Core deliverables |
|--------|------|-------------------|
| `proj0` | 2048 mechanics (staged) | Correct `GameLogic` tile moves & tilts |
| `proj0_hardmode` | 2048 mechanics (compact) | All-direction `tilt` |
| `proj1a` | ADTs | `LinkedListDeque61B` |
| `proj1b` | ADTs + audio | `ArrayDeque61B`, `Maximizer61B`, `GuitarString` |
| `proj2a` | NGrams data + server | `TimeSeries`, `NGramMap`, plotting handlers |
| `proj2b` | WordNet + NGrams | Hyponym queries + ranking in `HyponymsHandler` |
| `proj3` | Game/world | `World` + `Main` + tile-based rendering |

---

*This overview is inferred from file names, skeleton TODOs, and test classes in this repository. For grading details, follow your course’s official specification PDF.*
