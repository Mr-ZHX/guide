# SICP (NJUCS) — Project 4: Scheme Interpreter (English Lab Manual)

This English lab manual documents the **Scheme Interpreter project** implemented in `SICP/pro04`.
It focuses on the interpreter core and the complete set of Problems specified in `pro04`:
`Problem 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, EC` and optional items.

Note: In the current repository snapshot, `pro04` is the only part that contains explicit “Problem / BEGIN PROBLEM” placeholders and a corresponding test + question suite.
Other folders under `SICP/` (e.g. `hw*`, `lab*`, `pro01..pro03`) exist but do not include the same English Problem specification in this workspace.

---

## Table of Contents

1. Overview
2. Project Goals
3. Repository Files
4. Interpreter Design (Big Picture)
5. Required Components (Problems 1–14 and EC)
6. Scheme Homework Functions (Problems 15–16 and optional)
7. Special Forms and Tail Calls
8. Running the Interpreter and Tests
9. Testing Strategy and Debugging Tips

---

## 1. Overview

The goal of **Project 4** is to implement a working subset of a Scheme interpreter.
Scheme evaluation is performed by repeatedly applying rules for:

1. **Evaluating expressions** in an **environment**
2. **Applying procedures** to argument values
3. Handling Scheme’s **special forms** (e.g., `define`, `lambda`, `if`, `and`, `or`, `cond`, `let`)

This project is based on the “read–eval–print loop” (REPL) concept and the core interpreter structure from the course materials.

---

## 2. Project Goals

After completion, your interpreter should be able to:

- Correctly evaluate Scheme expressions including:
  - self-evaluating values
  - symbol lookup via environments
  - call expressions (operator + operand evaluation + procedure application)
- Implement procedure application for:
  - **built-in procedures**
  - **lexically scoped** `lambda` procedures
  - **dynamically scoped** `mu` procedures
- Support special forms:
  - `define` (both variable define and procedure define)
  - `quote`
  - `lambda`
  - `if`
  - `begin`
  - short-circuit `and` / `or`
  - `cond`
  - `let`
  - optional: `define-macro`, `quasiquote`/`unquote` (present in the framework; macro may be extra credit)
- Support **tail recursion optimization** for proper tail calls (EC)
- Pass the provided tests in:
  - `SICP/pro04/tests/*.py`
  - `SICP/pro04/tests.scm`

---

## 3. Repository Files (pro04)

Key files:

- `scheme.py`
  - Implements the REPL / interpreter entry point and global environment initialization.
- `scheme_eval_apply.py`
  - Implements `scheme_eval`, `scheme_apply`, `eval_all`, and optional tail-call optimization (EC).
- `scheme_forms.py`
  - Implements special forms in `SPECIAL_FORMS`.
- `scheme_classes.py`
  - Defines environment frames and procedure classes (`BuiltinProcedure`, `LambdaProcedure`, `MuProcedure`, …).
- `questions.scm`
  - Scheme-level homework functions (Problems 15, 16, and optional questions).
- `tests/*.py` and `tests.scm`
  - Automated tests (okpy / ok interface style).
- `proj04.ok`
  - The suite configuration used by the course test harness.
- `Makefile`
  - Convenience packaging / local ok run (requires okpy assets as referenced by the Makefile).

---

## 4. Interpreter Design (Big Picture)

The interpreter evaluates expressions with these major steps:

### 4.1 Environments (Frames)

- A **Frame** is a mapping from **symbols** (strings) to **values**.
- Each frame has a possible **parent frame**, forming a chain up to the global frame.
- `lookup(symbol)` searches current frame first, then parent frames.

### 4.2 Expressions

Expressions are evaluated based on their type:

- Atomic expressions:
  - numbers / booleans / strings / `nil` → self-evaluating
  - symbols → looked up in the environment
- Non-atomic expressions (combinations):
  - if the first element is a symbol and that symbol is in `SPECIAL_FORMS`, dispatch to the special form handler
  - otherwise treat as a call expression:
    - evaluate operator
    - evaluate operands (arguments)
    - apply the operator as a procedure to the evaluated arguments

### 4.3 Procedures

- **Built-ins**: implemented directly in Python (`BuiltinProcedure`), usually calling a provided Python function.
- **Lambda**: lexical scope
  - when called, creates a child frame whose parent is the frame where the lambda was defined.
- **Mu**: dynamic scope
  - when called, creates a child frame whose parent is the frame where `mu` is called.

### 4.4 Special Forms

Special forms evaluate some parts differently:

- `define` binds values to names without treating `define` as a normal function call
- `quote` returns raw data without evaluation
- `if` chooses a branch based on evaluated predicate
- `and` / `or` short-circuit: later expressions may not be evaluated
- `cond` chooses the first matching clause
- `let` creates a new child environment with evaluated bindings

---

## 5. Required Components (Problems 1–14 and EC)

The Python files contain `# BEGIN PROBLEM <n>` / `# END PROBLEM <n>` sections.
These correspond to the implementation tasks.

### Problem 1 — Environment Frame Basics (`scheme_classes.py`)

Implement:

- `Frame.define(symbol, value)`
- `Frame.lookup(symbol)` that searches current frame then parents

Requirements:

- Updating a binding in the same frame changes the value.
- Missing symbols should raise a `SchemeError`.

### Problem 2 — Applying Built-in Procedures (`scheme_eval_apply.py`)

Implement:

- `scheme_apply` behavior when `procedure` is a `BuiltinProcedure`.

Requirements:

- Convert Scheme list `args` to a Python argument list.
- If `procedure.expect_env` is set, pass `env` as an extra argument to the Python function.
- Catch argument count mismatches via `TypeError` and raise `SchemeError('incorrect number of arguments')`.

### Problem 3 — Evaluating Call Expressions (`scheme_eval_apply.py`)

Implement:

- In `scheme_eval`, for ordinary combinations (not special forms):
  - evaluate operator
  - evaluate operands (in order)
  - call `scheme_apply(operator, operands, env)`

Correctness note:

- Your interpreter should raise `SchemeError` for malformed lists (e.g., `(1)`).

### Problem 4 — `define` for Variable Binding (`scheme_forms.py`)

Implement:

- `(define <name> <expression>)`
- Evaluate `<expression>` and bind it to `<name>` in the current environment.

Returns:

- The symbol name `<name>` (as specified by the tests).

### Problem 5 — `quote` (`scheme_forms.py`)

Implement:

- `(quote <expression>)` returns the literal structure of `<expression>` without evaluating it.

### Problem 6 — Evaluating Expression Sequences (`scheme_eval_apply.py`)

Implement:

- `eval_all(expressions, env)`:
  - evaluate each expression in order
  - return the last expression’s value
  - if the list is empty, return `None` (undefined).

Must not mutate the input list.

### Problem 7 — `lambda` Procedure (`scheme_forms.py`)

Implement:

- `(lambda (<formals> ...) <body> ...)` returning a `LambdaProcedure`.
- Ensure the returned procedure captures the current environment (for lexical scoping).

### Problem 8 — Child Frame Construction (`scheme_classes.py`)

Implement:

- `Frame.make_child_frame(formals, vals)`

Requirements:

- Both `formals` and `vals` are Scheme lists represented by `Pair`.
- Bind each formal to its corresponding argument value in a new child frame.
- Raise a `SchemeError` if arity mismatches.

### Problem 9 — Applying `LambdaProcedure` (`scheme_eval_apply.py`)

Implement:

- In `scheme_apply`, when `procedure` is a `LambdaProcedure`:
  - create a child frame using `procedure.env.make_child_frame(formals, args)`
  - evaluate the lambda body with `eval_all`

### Problem 10 — Procedure Define Sugar in `define` (`scheme_forms.py`)

Implement the variant:

- `(define (<name> <params> ...) <body> ...)`

Requirements:

- Validate that the signature contains only symbols for parameter names.
- Bind `<name>` to a newly created `LambdaProcedure`.
- Return `<name>` (as tests expect).

### Problem 11 — Applying `MuProcedure` (`scheme_eval_apply.py`)

Implement:

- In `scheme_apply`, for `MuProcedure`:
  - create the child frame from the **calling environment** (`env.make_child_frame(...)`)
  - evaluate the body with `eval_all`

This implements dynamic scope behavior.

### Problem 12 — Short-Circuit `and` / `or` (`scheme_forms.py`)

Implement:

- `(and <test> ...)`
- `(or <test> ...)`

Requirements:

- Short-circuit:
  - `and` stops at the first falsey expression and returns `#f`/falsey value
  - `or` stops at the first truthy expression and returns that value
- `(and)` with no operands returns `#t`, `(or)` with no operands returns `#f`.
- Ensure correct behavior even when side-effects (mutation/printing) occur in later expressions.

### Problem 13 — `cond` (`scheme_forms.py`)

Implement:

- `(cond (<test> <expr> ...) ... (else <expr> ...))`

Rules:

- Evaluate clauses in order.
- The first clause whose `<test>` is true is selected.
- If a matching clause has no `<expr>`, return the test value.
- `else` must be the last clause.

### Problem 14 — `let` (`scheme_forms.py`)

Implement:

- `(let (<binding> ...) <body> ...)`

Where each binding is `(name expression)`.

Algorithm:

1. Evaluate each binding expression in the **current environment**
2. Create a new child environment frame extending the current environment
3. Bind names to evaluated values
4. Evaluate body expressions sequentially in the new environment

### EC — Proper Tail Recursion (`scheme_eval_apply.py`)

Implement tail-call optimization:

- When an expression is in **tail position**, return an “unevaluated” wrapper so evaluation can continue without growing Python recursion.

In this repository, the tail call optimization section is marked:

- `optimize_tail_calls` and `*** YOUR CODE HERE ***` (EC).

---

## 6. Scheme Homework Functions (Problems 15–16 and optional)

The file `questions.scm` contains Scheme functions to implement in the interpreter.

### Problem 15 — `enumerate`

Signature:

- `(enumerate s)` returns a list of two-element lists:
  - each element is `(index value)` for elements of `s`

### Problem 16 — `merge`

Signature:

- `(merge inorder? list1 list2)` merges two ordered lists.

Rules:

- If `inorder?` is true, the merged output list is ordered accordingly.
- Produces a combined list by repeatedly selecting the smaller head element (according to the comparator).

### Optional Problem 1 (in `questions.scm`) — `let-to-lambda`

Goal:

- Convert all `(let ...)` expressions into equivalent lambda applications.

### Optional Question 21 (in `questions.scm`) — Turtle Graphics “hax”

Goal:

- Draw a required image using the interpreter’s turtle graphics support.

---

## 7. Special Forms and Tail Calls (How to Reason About Them)

When implementing special forms, remember:

- Normal call expressions evaluate all operands eagerly before applying the procedure.
- Special forms may:
  - evaluate fewer operands
  - evaluate operands in a different order
  - skip evaluation entirely (short-circuit)

For tail recursion:

- Proper tail calls ensure the interpreter can evaluate deep recursion without exhausting the Python call stack.
- The project’s EC section uses the `Unevaluated` mechanism: instead of returning a final value immediately, tail-position evaluation returns a wrapper indicating the next expression to evaluate.

---

## 8. Running the Interpreter and Tests

### 8.1 Run the Scheme Interpreter REPL

From inside `SICP/pro04/`:

```bash
python3 scheme.py
```

This starts a Scheme REPL implemented by:

- `scheme.py`
- `read_eval_print_loop(...)`
- `create_global_frame()` which installs built-ins and procedures like `eval` and `apply`.

### 8.2 Run Test Suite

The project includes:

- `proj04.ok` (suite configuration)
- `tests/*.py` (python-based unit tests)
- `tests.scm` (Scheme-based tests)

The included `Makefile` shows a workflow that copies `ok` and `scheme` and then runs okpy locally:

```bash
make
```

To run tests with okpy directly (if okpy is installed in your environment), follow the `proj04.ok` configuration and typical okpy usage:

- Run the suite specified by the `.ok` file.
- Optionally select a subset of cases for faster iteration.

If you get stuck due to environment/tooling, use the REPL to manually check expressions from `tests.scm` until behavior matches expectations.

---

## 9. Testing Strategy and Debugging Tips

Recommended workflow:

1. Start with small “Problem <n>” doctest examples by directly running `python3` and importing the relevant modules.
2. Use `tests/01.py`, `tests/02.py`, etc. as references for expected behavior.
3. When special forms fail:
   - verify `SPECIAL_FORMS` dispatch logic in `scheme_eval`
   - confirm the number and order of evaluations in each form
4. When scoping fails:
   - check that `lambda` uses lexical scoping via `procedure.env.make_child_frame`
   - check that `mu` uses dynamic scoping via `env.make_child_frame`
5. For tail recursion EC issues:
   - validate that the interpreter returns `Unevaluated(...)` only in tail position
   - evaluate a deep recursive Scheme call and confirm no Python recursion error occurs

---

End of manual.

