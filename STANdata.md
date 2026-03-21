# CS246 Baseline — Project Overview (English)

This repository is the official **`cs246-baseline`** starter for **[Stanford CS246 — Mining Massive Data Sets](https://web.stanford.edu/class/cs246/)**. It is **not** split into multiple `lab1` / `lab2` folders: the **whole tree is one Gradle/Java template** used as the starting point for **Hadoop-based programming assignments** in the course.

This document explains **what the baseline provides** and **what you are expected to do with it** at a high level. It does **not** teach MapReduce or Gradle. It does **not** reproduce student write-ups.

---

## What is in the framework (single package)

| Piece | Role |
|--------|------|
| **`build.gradle`** | Java 8, **Hadoop 2.7.3** client dependencies, IDE plugins (**Eclipse** / **IntelliJ**), and **run tasks** (e.g. `runWordCount`) |
| **`src/main/java/.../WordCount.java`** | A complete **word-count** **MapReduce** job (**Mapper** + **Reducer** + **`ToolRunner`** driver) |
| **`data/pg100.txt`** | Sample **text input** (Project Gutenberg–style corpus) for local runs |
| **`gradlew` / wrapper** | Build without a global Gradle install |
| **`src/main/resources/log4j.properties`** | Logging configuration |

There are **no** separate assignment directories inside the repo; **each CS246 homework** typically asks you to **add new Java classes** and **new `JavaExec` tasks** in `build.gradle`, following the **`runWordCount`** pattern described in `README.md`.

---

## Functional goal of the included example

**`WordCount`** reads **line-oriented text** from an HDFS/local path passed as **argument 1**, tokenizes each line on whitespace, emits **(word, 1)** from the mapper, and sums counts in the reducer, writing **(word, count)** to the output path (**argument 2**).

**`./gradlew runWordCount`** (from the project root) **deletes** a previous **`output/`** directory, then runs the job with **`data/pg100.txt`** → **`output/`**.

**Outcome:** You can verify the toolchain (compile, run MapReduce locally or in the environment your course uses) before implementing **other** jobs required by CS246.

---

## What you do beyond the baseline (course-dependent)

- **Implement** additional MapReduce (or related) programs **in new `.java` files** under `src/main/java/...`.
- **Register** each program with a **`JavaExec`** task in **`build.gradle`** (copy `runWordCount`, change `main` and `args`).
- **Use** datasets and specifications from **CS246 handouts** (not shipped in this baseline).

The root **`README.md`** notes that a **full cluster** is not required for all work (see course **Piazza**/FAQ links there).

---

## Summary

| Question | Answer |
|----------|--------|
| How many “labs” are in the repo? | **None as separate folders** — one **baseline project** |
| What is the baseline for? | **Starting Hadoop MapReduce assignments** in **CS246** |
| What works out of the box? | **Word count** demo + **Gradle** run task + sample **input** |

---

*Actual homework numbering, deadlines, and grading are defined on the CS246 course site, not in this repository.*
