# EPFL CS-550 (Formal Verification) — Lab Framework Overview

This document summarizes **only what exists in this repository** under `cs550/`: numbered labs in `labs/lab1` … `labs/lab5`, the semester **project** materials in `project/`, and ancillary files (`README.md`, `labs/presentation.md`, `lectures/`). It is for readers new to the course: **what each piece is**, **what you must deliver or achieve**, and **which tools** are involved. It **does not** give solution strategies, proof sketches, or step-by-step algorithms.

**Course context (from root `README.md`):** CS-550 is EPFL’s *Formal Verification* course (Stainless, logic, proof assistants, projects). Labs are group work; grading also includes midterm and a final project.

**What is *not* a lab in this clone:** The root syllabus references weekly `exercises/` folders and `past-exams/`; those paths are **not present** in the checked-in tree here. The `lectures/lec16-last/*.scala` files are **lecture examples**, not assignment handouts.

---

## Tooling implied by the lab code

- **Scala 3** projects using **`scala-cli`** (and **`sbt`** for Lisa-related work as noted in Lab 1).
- **Stainless** (EPFL LARA) for verified Scala in Labs 1–2 and for termination/basic checks on Lab 3’s Scala code.
- **Lisa** interactive theorem prover (Scala-embedded) for Labs 4–5.
- **Java 17** for running Stainless (per Lab 1 README).

---

## Lab 1 — `labs/lab1/` (Introduction to Stainless)

**Goal:** Learn the Stainless workflow and split work between **machine-checked proofs** and **pure functional implementations** over small domains.

**Part 0 — Tutorial (`src/Tutorial.scala`):** Complete the official Stainless tutorial exercises in the starter file: implement and verify the listed recursive functions and lemmas (including list operations such as insert/sort-related specs). Extend the final `sInsert` specification with the **additional postcondition** required in the handout so Stainless still proves the strengthened contract.

**Part 1 — Sublists (`src/Sublist.scala`):** The `sublist` relation on lists is given. You supply **proofs** for a fixed list of lemmas (reflexivity, transitivity, antisymmetry, and related properties—**eleven** in total). You may **not** change the definition of `sublist` or the **statements** of the lemmas.

**Part 2 — Boolean formulas (`src/BooleanAlgebra.scala`):** Implement operations on propositional formulas represented as an ADT (variables, connectives, constants), including evaluation, substitution, negation normal form, variable extraction, and validity checking. Implement the parallel API for **and-inverter graphs (AIGs)** and a semantics-preserving translation from formulas to AIGs. Behavior is constrained by **`test/Tests.scala`**.

**Success criteria:** Stainless verifies the tutorial and sublist files; `scala-cli test .` passes for the Boolean-algebra tests.

---

## Lab 2 — `labs/lab2/` (Arithmetic and a communication protocol)

**Goal:** Reason in Stainless about **inductive data** (Peano-style naturals) and a small **stateful protocol** model.

**Part 1 — Arithmetic (`src/Arithmetic.scala`, with bundled `GodelNumbering.scala`):** Prove a lemma about exponentiation (`powMul`). Define a signed integer type `ZZ` (sign + magnitude over `Nat`) with the required invariant, then define **addition** and **multiplication** on `ZZ` and prove **commutativity** and **associativity** for both operations via the provided lemma stubs.

**Part 2 — Protocol (`src/SimpleProtocol.scala`):** The network/endpoints and finite-iteration message exchange are **already implemented**. You prove **seven** correctness/optimality-style lemmas about `messageExchange` without changing existing definitions or lemma signatures (auxiliary lemmas allowed if also proved).

**Success criteria:** Stainless accepts `Arithmetic.scala` (with `GodelNumbering.scala`) and `SimpleProtocol.scala`.

---

## Lab 3 — `labs/lab3/` (FOL semantics and resolution checking)

**Goal:** Build (in Scala, Stainless-checked) the **semantic evaluation** layer for a first-order logic representation and a **resolution proof checker**, then use it on a classic logic puzzle encoding.

**Semantics (`src/Interpretation.scala`):** Implement quantifier checks over finite domains, term evaluation, and formula evaluation under an interpretation, respecting variable **shadowing**. Must terminate on finite `Domain.elements`. Tests include finite fields (`FiniteField.scala`) and other small domains (`test/InterpretationSuite.scala`).

**Resolution pipeline (`src/Resolution.scala`):** Implement satisfiability-preserving transformations from formulas to a clausal form suitable for resolution: negation normal form, Skolemization, prenex Skolem form (matrix for universal closure), then CNF as `List[Clause]`. Implement **single-step** and **full proof** checking for resolution derivations (`Assumed` vs `Deduced` steps with explicit instantiations).

**Dreadsbury Mansion (`src/MansionFragments.scala`):** Complete short resolution **proof fragments** that the checker accepts: one fragment establishes a named character’s innocence; another completes the culprit proof using supplied prelude clauses from `Mansion.scala`. Runnable entry points are described in the README for interactive checking.

**Success criteria:** `scala-cli test .` passes; Stainless accepts the sources; proof fragments validate under the checker.

---

## Lab 4 — `labs/lab4/` (First-order proofs in Lisa)

**Goal:** Use the **Lisa** proof assistant (Scala) to write **explicit, machine-checked** proofs of first-order sequents—contrasting with Stainless’s SMT-backed verification style described in the handout.

**Work product:** Single file **`Lab04.scala`**. Starter content includes examples; you **complete all theorem proofs** so they check in green when running `scala-cli .`. The last theorems in the file are flagged as harder. Restrictions: do **not** use the `Tableau` or `Goeland` tactics (per README).

**Intended skills:** Terms, formulas, sequents, and basic Lisa tactics (hypothesis, weakening, propositional steps, quantifier rules—see Lisa manual referenced in README).

---

## Lab 5 — `labs/lab5/` (Lattices and automation in Lisa)

**Goal:** Prove elementary **lattice laws** in Lisa, then implement a **custom proof tactic** that automates a subclass of lattice inequations.

**Manual proofs:** Establish five named theorems (absorption, bounds, commutativity, associativity-style properties as listed in the README) using axiom instantiations and propositional steps—**no** heavy quantifier reasoning expected.

**Automation:** Implement a **proof-producing** variant of **Whitman’s decision procedure** for lattice inequations, following the **skeleton** in `Lab05.scala` (one case is worked as an example). The handout explains the high-level case split; your code must discharge obligations inside Lisa’s tactic API.

**Success criteria:** `scala-cli run .` reports the theorems/tactic behavior as expected when complete.

---

## Semester project materials — `project/` (not numbered like Lab 1–5)

**Role:** This folder supports the **final group project** and related written deliverables—not a weekly coding lab.

**Contents (in-tree):** `README.md` (topic choice, background paper, abstract, review expectations, Moodle links), `template.tex`, and `biblio.bib` as optional LaTeX scaffolding for the paper review.

**What students do (high level):** Agree on a verification-related project with staff; select and summarize a **background paper**; submit abstract and later a **short review**; carry out the project with code/results/presentation per course policy in the root `README.md`.

---

## `labs/presentation.md`

**Role:** Guidance for **oral presentation** structure and style for **student projects**—not a programming assignment.

---

## `lectures/lec16-last/`

**Role:** Small **Scala examples** (e.g. sorting-style fragments) bundled with lecture material; **no** lab README or submission target in this directory.

---

## What this overview omits

- **Installation commands**, solver flags, Moodle URLs, and deadlines (see each `README.md`).
- **Detailed transformation rules** for Lab 3, **Whitman case analysis**, and **Lisa tactic recipes**—those are instructional content in the handouts, not repeated here.
