# Compiler Project (MIPS-like CMM Compiler): English Experiment Descriptions (Total)

This document consolidates the required tasks for the `compiler` project. It is written in English and focuses on *experimental requirements*, *functional goals*, and *what needs to be implemented*. It does not copy any “completed report” content.

---

## Common Project Requirements (applies to all experiments)

### Experimental Requirements
- Use Linux to run the project.
- Keep the project structure requirements described by the repository:
  - The `Code/` directory contains flex/bison and C sources plus a `Makefile`.
  - Use the provided `Code/Makefile`. Besides editing/adjusting some non-critical pseudo targets, do not restructure files.
  - Do not create new subdirectories (e.g., `include/`, `src/`) inside `Code/`.
  - Avoid unreasonable include relationships (e.g., including one `.c` into another `.c`).
- Test files are located in `Test/` and must use the suffix `.cmm`.
- After building the compiler, replace the parser executable in `parser/` with your built binary (so the test harness uses it).
- You must fill in/replace `report.pdf` with your own report content for submission (this document does not reproduce any report content).

### Main Pipeline Functions (high level)
- Lexical analysis (flex): tokenization and lexical error reporting.
- Syntax analysis (bison): parsing according to the provided grammar and syntax error reporting.
- Semantic analysis (C): type checking and symbol/type consistency checks.
- Intermediate code generation: generate an intermediate representation (IR) for the program.
- Translation to target assembly (MIPS-like): convert IR to assembly output.

---

## Experiment 1: Lexical Analysis (Tokenization + Lexical Error Handling)

### Experimental Requirements
- Implement/verify the lexical analyzer using `flex` in `Code/lexical.l`.
- The scanner must:
  - Recognize CMM-like tokens used by the grammar (e.g., keywords, operators, delimiters).
  - Recognize numeric literals including:
    - decimal integers,
    - octal integers,
    - hexadecimal integers,
    - floating-point literals including exponent/various formats.
  - Handle comments:
    - `/* ... */` style comments,
    - `// ...` line comments.
  - Detect and report lexical errors according to the required error formats (as indicated by the `Error type ...` outputs in `lexical.l`).

### Functions (Modules to Understand/Touch)
- `Code/lexical.l`
  - Token rules for:
    - keywords (`if`, `else`, `while`, `struct`, `return`, `int`, `float`, ...),
    - identifiers,
    - delimiters (`; , ( ) [ ] { } .`),
    - operators (`+ - * /`, relational ops, logical ops `&& ||`, assignment `=`, unary `!`).
  - Numeric literal rules:
    - `int`, `int8` (octal-like), `int16` (hex),
    - floating forms used by the grammar.
  - Lexical error rules:
    - illegal hex/octal/number formats,
    - illegal identifiers/illegal tokens,
    - unterminated block comments,
    - mysterious characters.

### What Needs to Be Implemented
- Ensure the lexical analyzer outputs the correct token types and carries the correct semantic attributes (e.g., integer/float values, identifier strings).
- Ensure lexical errors are detected *at the lexical stage* and reported with the required message style (line numbers included).
- Ensure comment handling works correctly and unterminated/invalid comment sequences trigger the specified lexical/syntax error behavior.

---

## Experiment 2: Syntax Analysis (Parsing + Syntax Error Handling)

### Experimental Requirements
- Use `bison` to implement the parser according to `Code/syntax.y`.
- The parser must:
  - Build a parse tree / AST structure using the `add_node(...)` actions (as shown in `syntax.y`).
  - Correctly parse the CMM-like language constructs supported by the grammar, including:
    - external definitions (function declarations/definitions, global variables, struct declarations),
    - function definitions with parameter lists,
    - compound statements,
    - statements (`return`, `if/else`, `while`, expression statements),
    - expressions including arithmetic, relational, logical operators,
    - array indexing and array declarator syntax,
    - struct type usage and field access via `.`.
  - Implement syntax error handling / recovery according to the provided `yyerror` and `error ...` productions.

### Functions (Modules to Understand/Touch)
- `Code/syntax.y`
  - Grammar rules for:
    - `Program`, `ExtDefList`, `ExtDef`, `Specifier`, `StructSpecifier`,
    - `VarDec`, `FunDec`, `VarList`, `ParamDec`,
    - `CompSt`, `DefList`, `StmtList`, `Stmt`,
    - `Exp`, `Args`.
  - Locations/line tracking: `%locations` and usage of `@1.first_line`.
  - Error printing:
    - syntax error message output controlled by `yyerror(...)` and `fprintf(stdout, "Error type B ...")` patterns.

### What Needs to Be Implemented
- Ensure the grammar actions correctly:
  - construct the tree nodes with correct kinds (`SYNTAX_TYPE`, etc.),
  - propagate children pointers so that later semantic analysis can traverse the AST.
- Ensure syntax errors are reported consistently (line number and required error message style).

---

## Experiment 3: Semantic Analysis (Symbol Table + Type Checking)

### Experimental Requirements
- Perform semantic checks over the AST using the modules in `Code/semantics.c` and `Code/symbol.c`.
- The semantic analyzer must detect semantic errors indicated by the repository’s error outputs, such as:
  - redefined variables/fields/functions,
  - duplicated names,
  - inconsistent declarations/definitions of functions/structs,
  - undefined functions,
  - illegal assignments / type mismatches,
  - illegal use of struct types / struct field access.
- The semantic analyzer must also assign/check types for expressions and declarations:
  - int, float, struct types,
  - arrays with dimensions,
  - function parameter types and return types.

### Functions (Modules to Understand/Touch)
- `Code/semantics.c` / `Code/semantics.h`
  - `semantics_program`, `semantics_extdeflist`, `semantics_extdef`, ...
  - expression/type checking functions such as `semantics_exp`, `semantics_assignop`, `semantics_args`.
  - struct-related type resolution functions such as:
    - `semantics_specifier`,
    - `semantics_structspecifier`,
    - `semantics_tag`, `semantics_opttag`.
- `Code/symbol.c` / `Code/symbol.h`
  - symbol table management:
    - `symbol_init`, `add_entry`, `add_symbol`,
    - find functions: `find_symbol`, `find_symbol_struct`, etc.
  - checks for duplicates/inconsistencies and emitting required semantic error messages.

### What Needs to Be Implemented
- Implement/verify correct symbol table insertion and lookup for:
  - variables (including array declarations),
  - functions (declaration/definition consistency),
  - struct definitions and fields.
- Implement/verify type checking in assignments and expressions.
- Ensure semantic error output matches the required error message categories and includes correct line numbers.

---

## Experiment 4: Intermediate Code Generation (IR)

### Experimental Requirements
- Translate the semantically checked AST into intermediate code (IR) represented by `intercode` structures.
- The generated IR must include (at least) the constructs required by the later translation stage:
  - labels and control flow (`LABEL`, `GOTO`, `IF ... GOTO ...`),
  - assignments and arithmetic operations,
  - function-related IR (`FUNCTION`, `ARG`, `PARAM`, `CALL`, `RETURN`),
  - memory/IO-like operations used by the language (e.g., `READ`, `WRITE`),
  - array/struct-related address computation needed for array and field access.

### Functions (Modules to Understand/Touch)
- `Code/translate.c` / `Code/translate.h`
  - `translate_program`, `translate_extdeflist`, ...
  - `translate_exp`, `translate_stmt`, `translate_cond`,
  - array/struct related helpers such as:
    - `struct_array_offset`,
    - `struct_array_type`,
    - `translate_array_struct1/2`.
- `Code/intercode.c` (and `intercode.h`, `intercode` printing logic)
  - `intercode_init`, `intercode_new`, `intercodes_add`, `intercode_print`,
  - IR printing for each IR kind (assign/add/sub/mul/div/if/goto/return/dec/arg/call/...).

### What Needs to Be Implemented
- Implement/verify correct IR generation for all language constructs covered by the grammar and semantic analysis:
  - control flow (`if/else`, `while`),
  - expression evaluation,
  - function calls and returns,
  - array indexing and struct field access.
- Ensure IR printing/output format matches the translator expectations.

---

## Experiment 5: Translation to Target Assembly (MIPS-like)

### Experimental Requirements
- Convert the generated IR into target assembly using the `mips` generator.
- The output assembly should be a correct translation of control flow and dataflow in the IR.
- IO operations in the language (e.g., `read`, `write`) must be mapped to the corresponding assembly stubs/instructions expected by the environment.

### Functions (Modules to Understand/Touch)
- `Code/mips.c` / `Code/mips.h`
  - core emission functions:
    - `mips_init`, `mips_exp`, `mips_assign`,
    - `mips_if`, `mips_return`,
    - `mips_arg`, `mips_call`, `mips_param`,
    - `mips_read`, `mips_write`, `mips_print`, `mips_generate`.
- `Code/out1.s` (example output) and related IR outputs in the project root:
  - These show what the translator is expected to produce (useful for understanding output formatting).

### What Needs to Be Implemented
- Implement/verify register/operand mapping for IR operands (`mips_reg`, `mips_res` helpers).
- Ensure:
  - jumps/labels are emitted correctly for conditional and loop control flow,
  - arithmetic and assignment operations preserve semantics,
  - function call conventions (ARG/PARAM/CALL/RETURN) are consistent.

---

## Experiment 6: End-to-End Testing + Submission Workflow

### Experimental Requirements
- Build the parser using the provided `Code/Makefile`.
- Run tests on the `.cmm` files in `Test/` using the built binary.
- Use `parser/` replacement:
  - the test harness expects the `parser` executable to be replaced with your built output.
- Ensure your implementation passes the full set of provided test inputs:
  - syntax/lexical/semantic error cases,
  - valid programs demonstrating arrays, structs, function calls, and IO.

### Functions / Artifacts (What to Use)
- `Code/Makefile`
  - `parser` build target.
- `Test/test1.cmm`, `Test/test2.cmm`, `Test/test3.cmm` (and any additional `.cmm` tests)
  - Each test checks a subset of language features and/or error cases.
- `parser/`
  - Replace the parser binary with your implementation so that tests run correctly.
- `report.pdf`
  - Replace with your own report content for submission (this document does not include it).

### What Needs to Be Implemented
- An end-to-end pipeline that works for all tests:
  1. input `.cmm` file,
  2. scan tokens + detect lexical errors,
  3. parse into AST + detect syntax errors,
  4. perform semantic analysis + emit semantic error messages,
  5. generate IR and translate to assembly for correct programs.

---

