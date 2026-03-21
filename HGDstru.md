# Data Structure Course Experiments — English Overview

This document describes the experiments under `data-structure/Experiments/` (also reachable as `data-structure\Experiments` on Windows). It is written for readers who are **not** familiar with the assignments: for each lab it states **what you are expected to do**, **what functionality the program demonstrates**, and **what “correct” means at a high level**. It does **not** give implementation recipes and does **not** include anyone else’s lab report.

The experiments are organized as `DSExp01` … `DSExp05` (with **two** distinct topics under `DSExp05`).

---

## DSExp01 — Stack (`DSExp01_Stack`)

### What you do
Use a **stack** as the central data structure to process **arithmetic expressions**: convert **infix** notation to **postfix**, then **evaluate** postfix for numeric inputs.

### What functionality is implemented (as reflected in the code)
1. **Input modes**: read expressions from a file (`samples.txt`) or type them interactively; optional **log file** (`log.txt`) that records stack states during conversion.
2. **Infix → postfix**: supports operators such as `+`, `-`, `*`, `/`, `^`, and parentheses; produces a space-separated postfix string.
3. **Postfix evaluation**: for purely numeric expressions, computes a floating-point result.
4. **Symbolic branch**: if the expression contains **letters**, the program treats it as **expression preparation / simplification mode** (it augments implicit multiplication between digit and variable, then prints postfix rather than a numeric value).

### Success looks like
Printed postfix forms and numeric results match expectations for valid inputs; optional logging shows consistent stack traces during conversion.

---

## DSExp02 — Binary Tree (`DSExp02_Tree`)

### What you do
Implement interactive operations on a **binary tree** built from user input, using **stacks** and **queues** where appropriate.

### What functionality is implemented
1. **Tree construction**: preorder input with `#` marking empty subtrees.
2. **Traversals**:
   - preorder, inorder, postorder — each with **recursive** or **iterative** variant;
   - **level-order** (breadth-first) traversal using a queue.
3. **Property check**: decide whether the tree is a **complete binary tree**.
4. **Lowest common ancestor (LCA)**: given two node values, report the nearest common ancestor’s value.

### Success looks like
Traversals print in the correct order; complete-tree judgment and LCA outputs match the tree you built.

---

## DSExp03 — Graph (`DSExp03_Graph`)

### What you do
Work with an **undirected graph** loaded from an edge list file (`data.txt`: pairs of vertex indices), using both **adjacency matrix** and **adjacency list** representations.

### What functionality is implemented
1. **Build graph**: create either an adjacency **matrix** or an **adjacency list** from the file.
2. **Convert**: transform matrix ↔ list while preserving the same graph.
3. **Depth-first search (DFS)**: run DFS from every start vertex; supports **recursive** and **iterative** DFS for **both** representations.
4. **Breadth-first search (BFS)**: BFS from every start vertex for **both** representations.

### Success looks like
Printed adjacency structures match the input edges; DFS/BFS visit sequences are produced for each starting vertex as defined by the program’s traversal rules.

---

## DSExp04 — Search: BST, Binary Search, Average Search Length (`DSExp04_Search`)

### What you do
Compare **binary search trees** built from different insertion orders, implement **binary search** on a sorted array, and measure **average search length (ASL)** for successful and unsuccessful searches in different settings.

### What functionality is implemented
1. **Two BSTs** initialized from files:
   - `order.txt` — values in sorted order;
   - `random.txt` — values in random order.  
   On each tree: **insert**, **delete**, **search**, **inorder sort output**.
2. **Binary search** on a sorted array built from `order.txt`.
3. **ASL statistics**:
   - for ordered vs random BST: successful search ASL and a modeled **failure** ASL;
   - for binary search: uses an auxiliary **decision tree** (`HalfNode`) built by splitting index ranges to estimate success/failure ASL.

### Success looks like
Search and sort operations behave like standard BST/binary-search semantics; printed ASL numbers are produced for comparison between the ordered BST, random BST, and binary-search structure.

---

## DSExp05a — Hashing & Consistent Hashing (`DSExp05_Hash`)

### What you do
Simulate **distributed server selection** using **hashing** and a **ring** of **virtual nodes** (a simplified consistent-hashing style assignment).

### What functionality is implemented
1. **Server pool**: maintain real server IDs (random IP strings at startup); each real server maps to multiple **virtual nodes** in a sorted map keyed by hash.
2. **Hash function**: fixed-string hashing (implementation uses an FNV-style mixing loop in the skeleton).
3. **Routing**: given a **client** string (e.g., client IP), compute its hash and map it to the **next** virtual node on the ring (`tailMap`), then resolve to the owning real server.
4. **Dynamic cluster**: **add** or **remove** servers and update virtual nodes accordingly.

### Success looks like
Queries consistently assign clients to servers; after add/remove, routing updates without manual renumbering of all clients (modulo the simplified model in the demo).

---

## DSExp05b — Red–Black Tree (`DSExp05_RedBlackTree`)

### What you do
Implement a **red–black BST** supporting insertion, deletion, and rebalancing, initialized from sorted data in `order.txt`, then exercised interactively.

### What functionality is implemented
1. **RB-tree operations**: insert (with BST placement + rebalance), search, delete (with fix-up), inorder traversal listing.
2. **Performance analysis**: compute **average successful search length** and **average unsuccessful search length** under the program’s counting model.
3. **Stress / debug mode** (optional menu branch): random batch insert/delete/search to validate robustness.

### Success looks like
The tree maintains search ordering; rebalancing keeps the red–black invariants as enforced by the skeleton’s rotation/recolor logic; traversal lists keys in sorted order.

---

## Summary Table

| Folder | Topic | Core ideas |
|--------|------|------------|
| `DSExp01_Stack` | Stack | Infix/postfix, evaluation, optional logging |
| `DSExp02_Tree` | Binary tree | Build, traversals, completeness, LCA |
| `DSExp03_Graph` | Graph | Matrix/list, conversion, DFS/BFS |
| `DSExp04_Search` | Search & ASL | BST shapes, binary search, average lengths |
| `DSExp05_Hash` | Hash / consistent hashing | Virtual nodes, ring routing, cluster changes |
| `DSExp05_RedBlackTree` | Balanced BST | RB insert/delete, rotations, ASL stats |

---

*End of document.*
