# BUPT Lab — CFG & PDA (Formal Languages) — Work Packages Overview (English)

This repository supports the **Formal Languages and Automata** course experiment at **BUPT** (北京邮电大学, **2023–2024 Spring**): **context-free grammars (CFG)** and **pushdown automata (PDA)**), implemented in **Python**.

The tree has **no** folders named `lab1`, `lab2`, etc. This document groups work **exactly by what the framework provides**: **`src/datastructure`**, **`src/algorithm`**, matching **`test/*.py`**, plus the interactive **`main.py`**.

It describes **what each part must accomplish**—not algorithms or proof details. It does **not** copy student reports (the root `README` mentions reports in **Release**; that material is out of scope here).

---

## Data structures (foundation)

### CFG module (`src/datastructure/cfg.py`) — covered by `test/test_cfg.py`

**Purpose:** Represent a **CFG** as **(N, T, P, S)** with productions in a normalized tuple form (single non-terminal on the left; right-hand side a tuple of symbols; **ε** represented as the empty string `""` in the implementation).

**Functionality:** **Validate** a grammar (disjoint terminals/nonterminals, start symbol in **N**, left-hand sides are single non-terminals, symbols on each side belong to **N ∪ T ∪ {ε}**). **Pretty-print** the grammar to a human-readable string.

**Outcome:** Tests in **`test_cfg.py`** check validity and string rendering for sample grammars.

### PDA module (`src/datastructure/pda.py`)

**Purpose:** Represent a **PDA** **(Q, Γ, δ, q₀, Z₀, F)** with **δ** as a map from **(state, input symbol or ε, stack top)** to sets of **(next state, stack string to push)**.

**Functionality:** **Validate** structural consistency (states, alphabet, stack symbols, start, finals). Expose whether the machine is treated as an **empty-stack (accept-by-empty-stack)** PDA in the sense used by the rest of the code (`is_epda()` in the full file).

**Outcome:** Used as input to conversion tests and the CLI.

---

## Algorithm 1 — Eliminate ε-productions (`algorithm/eliminate_epsilon_production.py`)

**Purpose:** Transform a CFG into an **equivalent** grammar **without ε-productions** (except possibly the special case for the start symbol if the empty string is in the language—behavior as defined by the implementation and tests).

**Tests:** **`test/test_eliminate_epsilon_production.py`**

**Outcome:** Given grammars with ε-rules, the output grammar matches the expected normal form in tests.

---

## Algorithm 2 — Eliminate unit productions (`algorithm/eliminate_unit_production.py`)

**Purpose:** Remove **unit productions** (A → B with B a single non-terminal), producing an equivalent grammar without them.

**Tests:** **`test/test_eliminate_unit_production.py`**

**Outcome:** Unit-free grammar equivalent to the input on the test cases.

---

## Algorithm 3 — Eliminate useless symbols (`algorithm/eliminate_useless_symbol.py`)

**Purpose:** Remove **non-generating** and **non-reachable** symbols (and productions that use them), yielding a **reduced** grammar covering only useful symbols from the start symbol.

**Tests:** **`test/test_eliminate_useless_symbol.py`**

**Outcome:** Smaller equivalent grammars on the provided examples.

---

## Algorithm 4 — Final-state PDA → empty-stack PDA (`algorithm/transfer_fpda_to_epda.py`)

**Purpose:** Convert a **final-state acceptance** PDA into an **empty-stack acceptance** PDA that recognizes the **same language**.

**Usage in project:** Called from **`main.py`** when the user chooses PDA→CFG and the input PDA is **not** already in EPDA form.

**Outcome:** Correct transformation consistent with downstream **PDA → CFG** construction.

---

## Algorithm 5 — Empty-stack PDA → CFG (`algorithm/transfer_epda_to_cfg.py`)

**Purpose:** Apply the standard construction from an **EPDA** to a **CFG** (state/stack non-terminals and productions as in the textbook construction).

**Tests:** **`test/test_pda_to_cfg.py`** (expects a concrete **CFG** equivalent to a sample **EPDA**).

**Outcome:** The generated **CFG** matches the reference grammar in the test.

---

## Integrated application (`src/main.py`)

**Purpose:** Small **menu-driven** program (not separately tested as a whole in the listed tests, but ties everything together):

1. **Option 1 — Simplify a CFG:** User types **N**, **T**, productions (`->`, alternatives with `|`), and **S**. The program runs **ε-elimination → unit elimination → useless-symbol elimination** and prints the result.
2. **Option 2 — PDA to CFG:** User types **Q**, **T**, **Γ**, transitions in the documented `(q,a,X) -> {(q', stack), ...}` style, **q₀**, **Z₀**, **F**. If the PDA is not EPDA, it is converted; then **EPDA → CFG**; then the same **CFG simplification pipeline** as option 1.

**Outcome:** End-to-end demonstration of all algorithms for interactive input.

---

## Summary table (framework ↔ tests)

| Component | Role | Automated tests |
|-----------|------|-----------------|
| `cfg.py` | CFG ADT + validity + print | `test_cfg.py` |
| `pda.py` | PDA ADT + validity | (used by `test_pda_to_cfg.py`) |
| ε-elimination | Grammar normalization | `test_eliminate_epsilon_production.py` |
| Unit elimination | Grammar normalization | `test_eliminate_unit_production.py` |
| Useless symbols | Grammar cleanup | `test_eliminate_useless_symbol.py` |
| FPDA → EPDA | Acceptance mode change | (via `main` + conversion chain) |
| EPDA → CFG | PDA equivalence to grammar | `test_pda_to_cfg.py` |
| `main.py` | CLI integration | — |

---

*Run `pytest` in the project root (with dependencies from `requirements.txt`) to execute the test suite.*
