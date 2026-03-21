# Compiling Principle Experiments — Labs Overview (English)

This repository (`Compiling-principle-experiment`) contains **two** self-contained C++ assignments under **`LexicalAnalyzer/`** and **`Parser/`**. There is **no** third lab folder in the tree. Specifications below are taken from each folder’s **`readme.md`** and the presence of **`LexicalAnalysis.cpp`** / **`SyntacticParser.cpp`**.

This document states **what each lab expects** and **what the program must do**. It does **not** teach algorithms or reproduce student reports, screenshots, or maintainer slogans.

---

## Lab 1 — Lexical analyzer (`LexicalAnalyzer/`)

**Goal:** Build a **lexer** for a small **C-like** source language (dialect is up to you, within the assignment constraints).

**Input:** A **`.txt`** file written by the user (convention in the docs: e.g. **`testCpp.txt`** next to the program) containing a short source program that satisfies:

- At least **50 non-space characters**
- Contains **legal identifiers**, **keywords**, **decimal integers**, and **operators**
- Contains **newlines** and **comments**: both **`/* … */`** (may span lines) and **`//`**
- Contains **at least one illegal identifier**
- Contains **extra spaces**

**Processing:**

1. **Preprocessing:** Remove comments, newlines, redundant spaces/tabs as specified, then **show** the resulting character stream and **save** it to a **new text file** in the **same directory** as the source (e.g. **`processed.txt`** / analogous name per implementation).

2. **Scanning:** Tokenize the preprocessed stream. For each **valid** token, record **(sequence number, category code, lexeme)** where the sequence number is the token’s position in the preprocessed stream. **Write** these triples to **another new text file** in that folder and **display** them.

3. **Errors:** **Illegal identifiers** are **not** listed in the normal token list; at the end, emit **error reports** giving the **token index** and the **illegal lexeme** (and show on console as in the handout).

**Token categories (example mapping from the handout):** keywords and symbols such as `void`, `main`, `int`, `cout`, `return`, parentheses, braces, `<<`, punctuation, `=`, `*`, `/`, **identifier (16)**, **decimal integer (17)**, etc. (full code table is in `LexicalAnalyzer/readme.md`).

**Outcome:** Preprocessed file + token list file + error listing for non-conforming identifiers, for the supplied test style.

---

## Lab 2 — LL(1) syntactic parser (`Parser/`)

**Goal:** Implement an **LL(1) parser generator/driver** for a **Type-2 (CFG) grammar** read from a file: compute **FIRST / FOLLOW**, build the **LL(1) parsing table**, and **simulate** table-driven parsing on user strings.

**Grammar file (TXT) constraints:**

- At least **two terminals** and **two non-terminals**
- At least **one production** whose right-hand side can be **empty**, written as **`@`** (stands for **ε**)
- At least **four productions** total; at least one RHS has **two or more symbols**
- At least **one** production is **neither purely left-linear nor purely right-linear**
- At least **one non-terminal** has **two or more alternative productions**
- **No redundant** rules; grammar must **generate a non-trivial language**

**Required outputs:**

1. For each **distinct right-hand-side string** appearing in productions, show the **FIRST** set of that string (each unique RHS string reported once).
2. For every **non-terminal**, show its **FOLLOW** set.
3. Print the **LL(1) parsing table** for the grammar.

**Interactive / batch behavior:** The user supplies a **string of terminals**; the program prints the **parse steps** (stack/input/configuration style as required) and concludes whether the string **is** or **is** **not** a sentence of the grammar. The assignment asks for **two** test strings: one **accepted**, one **rejected**.

**Framework code:** `SyntacticParser.cpp` defines structures for **rules**, **FIRST/FOLLOW sets**, and the **LL(1) table**, with functions for loading the grammar file (e.g. **`test1.txt` / `test2.txt` / `test3.txt`** in the repo), building the table, and running **syntactic analysis**.

**Outcome:** Correct FIRST/FOLLOW, a consistent LL(1) table, and correct accept/reject decisions with trace output for sample grammars.

---

## Repository map

| Path | Lab |
|------|-----|
| `LexicalAnalyzer/` | Lab 1 — lexer + preprocessor |
| `Parser/` | Lab 2 — LL(1) theory + table + parse |

---

*If your course uses different filenames or extra hidden tests, follow your instructor’s handout; this file reflects only what this repository contains.*
