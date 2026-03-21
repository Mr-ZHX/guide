# Data Structures Course (`DataStructure-ZNJ-SHU`) — English Lab Overview

This document summarizes the labs under `spring/` and `winter/` in this repository. It is written for readers who are **not** familiar with the course: for each experiment it states **what you must accomplish**, **what functionality to deliver**, and **what inputs/outputs or behaviors define success**, without implementation steps and without copying anyone’s lab report.

---

## Spring semester labs (`spring/`)

### Spring Lab 1 — `exp01-adjacent-matrix-weighted-directed-graph`
**Working name (from README):** Adjacency-matrix representation of a **weighted directed network** (non-negative weights), with extensions.

**What you do**  
Design and implement an adjacency-matrix class template for a weighted directed graph (“network”), analogous to an undirected-graph matrix class, including basic graph operations (e.g., add/remove vertices and arcs).

**What functionality you implement**
1. `CountOutDegree(v)` — out-degree of vertex `v`.
2. `CountInDegree(v)` — in-degree of vertex `v`.
3. `ShortestPath(v1, v2)` — shortest path between two vertices (single-source shortest paths on non-negative weights; the course notes mention a **heap-optimized Dijkstra** style approach for efficiency when selecting the next vertex).

**Success looks like**  
Basic graph edits behave correctly; degree counts match the matrix; shortest-path queries return correct distances/paths for valid inputs.

---

### Spring Lab 2 — `exp02-adjacent-list-weighted-undirected-graph`
**Working name:** Adjacency-list representation of a **weighted undirected network**, with MST-related extensions.

**What you do**  
Implement an adjacency-list class template for a weighted undirected graph, with basic operations (add/remove vertices and edges).

**What functionality you implement**
1. `CountDegree(v)` — degree of vertex `v`.
2. `ConnectedComponent()` — number of connected components.
3. **Minimum spanning trees:** validate **Kruskal** and **Prim** on connected graphs, and implement **at least one additional** MST algorithm (e.g., a “cycle-breaking / destroy-cycle” method—README mentions examples like 破圈法).
4. `hasUniqueMinTree()` — decide whether the graph has a **unique** minimum spanning tree (connected graph case).

**Success looks like**  
Component counts and MST weights/edges match expectations; uniqueness predicate matches theory on provided tests.

---

### Spring Lab 3 — `exp03-search`
**Working name:** Search algorithms—design, verification, and extensions.

**What you do**  
Complete three separate algorithmic tasks (split across `task01`, `task02`, `task03` sources in the skeleton).

**What functionality you implement**

1. **Smallest common element of three sorted arrays**  
   Given three ascending integer arrays, find the **smallest value** appearing in all three, and report its index positions in each array; if none exists, output `NOT FOUND`. Required worst-case time **O(n)** for total length \(n\).

2. **Median of two sorted sequences (length \(n\) each)**  
   Define the median as the element at position \(n\) in the merged sorted order of the two length-\(n\) sequences. Implement **two algorithms**; at least one must run in **O(log n)** worst case. (README notes an optional harder “general” variant for unequal lengths beyond the basic spec.)

3. **Binary search tree deletion—variant #3**  
   The textbook implements one deletion strategy; you must implement **deletion by moving the left subtree under the successor** (the “third” method). Then **benchmark** the textbook method vs. yours on multiple tests and compare search performance.

**Success looks like**  
Public I/O matches the sample formats; BST deletion variant passes functional tests; performance comparison produces measurable results.

---

### Spring Lab 4 — `exp04-sort`
**Working name:** Sorting—quick sort improvements and DNA classification.

**What you do**  
Two parts: improve quicksort pivot selection, and solve a DNA ranking problem.

**What functionality you implement**

1. **Improved quicksort**  
   Propose **1–2 pivot-selection strategies** so partitions are more balanced; compare against the baseline using experiments.

2. **DNA sorting by inversion count**  
   For DNA strings over `{A,C,G,T}`, compute each string’s **inversion number** (pairs of characters out of order). Sort strings by **ascending inversion count**; ties break by **lexicographic descending** order among strings with equal inversion counts. Use **your own sorting** (not a library sort routine for the core requirement—per typical course interpretation).

**Success looks like**  
DNA output matches the official sample ordering pattern; quicksort experiments show documented performance differences.

---

## Winter semester labs (`winter/`)

### Winter Lab 1 — `exp01-interview-arrangement`
**Working name:** Circular “resume picking” (Josephus-style) with two recruiters.

**What you do**  
Simulate picking items arranged in a **circle** labeled `1..N`:
- **Task 1:** Two people start at fixed positions and repeatedly count **K** steps counterclockwise and **M** steps clockwise, removing selected items until all are taken. If both select the same item, output it once. Output the removal order as pairs (first person’s pick, second person’s pick).

- **Task 2:** Same as task 1, but after each round (until finished), a **new item** is inserted before the first person’s next starting position (new IDs continue from `N+1`). The README warns some inputs may **never finish**; the repo includes a note/proof file about non-termination conditions.

**Extra in repo:** An **efficiency test** can time multiple task-1 inputs and write `output/result.csv`.

**Success looks like**  
Outputs match the sample formats; task 2 respects insertion rules; optional timing output is produced when enabled.

---

### Winter Lab 2 — `exp02-car-scheduling`
**Working name:** Train carriage scheduling with a stack (“T-shaped” rails).

**What you do**  
Model a classic **stack-based reordering** of train cars on a main track with a side spur (stack).

**Task 1:** Cars `1..n` arrive in order `1..n` on the left; determine whether they can exit on the right in a **given target permutation**, and print a valid sequence of moves if possible.

**Task 2:** Cars arrive on the left in a **given permutation** of `1..n`; determine whether they can exit on the right in sorted order `1..n`, and print moves if possible.

**Extra in repo:** Validation can enumerate small cases and relate counts to **Catalan numbers**; may save exhaustive success cases to `output/` (README warns factorial cost—keep `n` small).

**Success looks like**  
For feasible inputs, a move trace is printed; for infeasible inputs, a failure message; optional validation statistics match expected combinatorics for small `n`.

---

### Winter Lab 3 — `exp03-literature-assistant`
**Working name:** “Literary research assistant” — pattern statistics in a text file.

**What you do**  
Scan an English novel **once** and report, for each query word, **how many times** it occurs and **which line numbers** contain it. Words are defined as strings of letters and `-` only; lines are bounded (README: ≤ 120 chars per line assumption in the spec text).

**What functionality you implement**  
Efficient multi-pattern scanning over the novel; README notes **KMP-style `next` array** optimization and **overlapping matches** (e.g., pattern `aba` can match twice in `ababa`). Inputs/outputs are wired through `data/patterns.txt`, `data/text.txt`, and `output/result.txt` in this skeleton.

**Success looks like**  
Counts and line lists match the definition of “word” and overlap rules; single-pass requirement is met relative to the course grader.

---

### Winter Lab 4 — `exp04-binary-tree`
**Working name:** Binary tree extensions + a labeled “marked tree” counting problem.

**What you do**

1. **Swap subtrees**  
   Add a member `Revolute()` to a binary linked-tree class template that **swaps left and right subtrees at every node** (whole-tree reflection-like operation on structure).

2. **Marked binary tree path counts**  
   Nodes are labeled by a deterministic rule starting from root `(1,1)`; children of `(a,b)` are `(a+b,b)` left and `(a,a+b)` right. Given a target label `(a,b)`, compute how many **left** and **right** branches are taken from the root to reach it.

**Success looks like**  
Task 1 transforms the tree as specified; task 2 matches sample I/O (e.g., `(42,1) → 41 left, 0 right`; `(3,4) → 2 left, 1 right`).

---

## Quick reference

| Season | Folder | Topic summary |
|--------|--------|----------------|
| Spring | `exp01-...-directed-graph` | Weighted directed graph matrix + degrees + shortest paths |
| Spring | `exp02-...-undirected-graph` | Weighted undirected list + components + MST + unique MST |
| Spring | `exp03-search` | 3-way merge search, two-sequence median, BST delete variants |
| Spring | `exp04-sort` | Quicksort pivot study + inversion-based DNA sorting |
| Winter | `exp01-interview-arrangement` | Circular picking + dynamic insertions + optional timing |
| Winter | `exp02-car-scheduling` | Stack rail scheduling + Catalan validation |
| Winter | `exp03-literature-assistant` | Multi-pattern text statistics (KMP-style) |
| Winter | `exp04-binary-tree` | Subtree swap + rational-label path counting |

---

*End of document.*
