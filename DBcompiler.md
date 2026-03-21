# NEU Compiler Design (`neu-compiler-design`) — Lab Framework Overview (English)

This document describes **only what exists in this repository**: a **Java / Maven** project for a **compiler-principles course design** (Northeastern University, per `README.md`) with a **lexer** and three **parser front-end** variants, each able to drive **quadruple (three-address code) generation** for simple arithmetic expressions. It is written for readers who know nothing about the assignment: **what each part is for** and **what behavior is checked**, without implementation steps or student-style reports.

**In-tree sources:** `README.md`, `pom.xml`, `src/main/java/**`, `src/test/java/**`.

---

## Repository structure (no separate `lab/` folders)

The coursework is **partitioned by packages**, not by numbered lab directories:

| Package / area | Role |
|----------------|------|
| `lexer` | Lexical analysis: read source text, classify lexemes, emit token stream (and auxiliary tables). |
| `parser.LL1` | LL(1) predictive parsing with an **analysis table** built from **FIRST / FOLLOW / SELECT**; semantic actions produce **quadruples**. |
| `parser.LR` | Table-driven **LR-style** parsing (implementation is framed as LR(0) with an embedded action table; comments refer to SLR(1)); produces **quadruples**. |
| `parser.Recursive` | **Recursive-descent** parsing for the same expression language; produces **quadruples** and explicit **error reporting**. |

**Build / test:** Maven, Java 8, **JUnit 5** (`junit-jupiter-api` in `pom.xml`). Tests live under `src/test/java/lexer_test` and `src/test/java/parser_test`.

---

## Component 1 — Lexical analysis (`lexer/`)

**Purpose:** Implement a **scanner** that reads characters from a configured input file, groups them into tokens, and writes results to an output file.

**Main types (from source headers):**

- **`WordScanner`** — Orchestrates file I/O (`input.txt` → analyze → `output.txt`), maintains buffers for the character stream, accumulated **`Token`** list, and auxiliary lists for **keywords**, **identifiers**, **symbol/delimiter** strings, and **numeric constants**. Handles comment skipping at the line level where applicable.
- **`Token`** — Pairs a **numeric type code** with a **string value**; string form like `<type, value>` for logging/output.
- **`JudgeType`** — Supporting logic for token / character classification (used by the scanner).

**Expected functionality (inferred from tests and class comments):** Recognize literals such as integers, floating-point forms, and scientific-notation numbers (e.g. `2e-2`), along with other lexical categories exercised in `lexer_test.WordScanner` (run **one test at a time** when using real files—the test class warns that batch runs can interfere with shared files).

**I/O fixtures:** `src/main/java/lexer/file/input.txt` and `output.txt` are the working paths used by the scanner workflow.

---

## Component 2 — LL(1) syntax analysis (`parser/LL1/`)

**Purpose:** Parse **infix arithmetic expressions** over identifiers, literals, `+ - * /`, parentheses, and a terminating **`;`**, using an **LL(1) control program** and a predictive **parsing table**.

**Main pieces:**

- **`AnalysisTable`** — From a **context-free grammar** (configurable via `setGrammar`), compute **nonterminals / terminals**, **FIRST**, **FOLLOW**, and **SELECT** sets, test whether the grammar is **LL(1)-compatible**, and materialize the **parsing table** for inspection.
- **`LL1`** — Drives stack-based LL(1) parsing; integrates **semantic stack** operations; emits **`Quadruple`** records (operator, two operands, result temp).
- **`AnalysisTableItem`**, **`Quadruple`** — Table cell representation and intermediate-code tuples.

**Expected functionality (from tests):** Correct table construction for a valid grammar; detection / handling path for a **non–LL(1)** sample production; parsing strings such as `a+b;`, `a-b;`, `a*b;`, `a/b;`, and combined expressions like `(a+b-c)*d/e;`, with quadruple sequences matching the intended three-address breakdown.

---

## Component 3 — LR syntax analysis (`parser/LR/`)

**Purpose:** Parse the same style of **expression + `;`** language using a **precomputed LR parse table** (shift/reduce entries with embedded semantic action codes) and stacks for syntax and semantics.

**Main pieces:**

- **`LR`** — Implements the **LR control loop** (`LRControl`), maintains **quadruple** output, and can **reject** invalid inputs (tests include malformed strings).
- **`Quadruple`** — Same intermediate-representation role as in LL(1).

**Expected functionality (from tests):** Successful parses for `a+b;`, `a*b;`, `(a+b)*c;` with correct quadruple order; **failure** (boolean `false`) for clearly invalid input such as `aaa;`.

---

## Component 4 — Recursive-descent syntax analysis (`parser/Recursive/`)

**Purpose:** Parse the arithmetic language with **mutually recursive procedures** (one nonterminal per routine pattern), again producing **quadruples**.

**Main pieces:**

- **`Recursive`** — Entry `beginToAnalysis` loads the string, calls the start symbol routine, and enforces a **trailing semicolon**; throws on structural errors with distinct error messages (e.g. missing `;`, illegal leading token, unbalanced parentheses—covered by tests `ERR0`–`ERR2`).
- **`Quadruple`** — Intermediate code tuples.

**Expected functionality (from tests):** Same family of valid expressions as other parsers; **exceptions** for missing semicolon, bad first character, and unmatched parenthesis.

---

## Cross-cutting theme: intermediate code

All three parsers target **quadruples** of the form **(operator, arg1, arg2, result)** for binary operations, using temporary names like `t1`, `t2`, … Tests assert **equality** of emitted quadruples against expected sequences for representative inputs.

---

## What this overview omits

- **Grammar productions, table entries, and automaton matrices** (they are part of the solution code, not a neutral spec).
- **README** grade commentary and **author attributions** in test headers (not instructional requirements).
- **IDE** metadata (`.idea/`) and **`target/`** build outputs.

If the course publishes a separate PDF handout, use that for official deadlines and report rules; this repository does not include such a document.
