# CourseDesignDBMS — Experiment / Project Overview (English)

`CourseDesignDBMS` is a **database course design project**: a small **information system** for managing **database-course design** offerings at a university. It serves **three kinds of users**—**students**, **teachers**, and **anonymous visitors**—and is backed by **MySQL** with a **Spring Boot** REST API (`CourseDesignDBMS` module).

This repository does **not** split work into numbered `lab1` / `lab2` folders. The work is one **integrated capstone**, naturally broken into **design phases** and **role-based features** below.

This document explains **what the project is for** and **what must exist functionally**. It is **not** an implementation guide. It **omits** team rosters, screenshots, and narrative from the design write-up in `README.md` that belong in a submitted report.

---

## Phase A — Requirements analysis

**Goal:** Clarify stakeholders and behaviors.

**What you produce:** A short **requirements** document: who uses the system (students, teachers, guests) and what each role needs to do (authentication, topic selection, grading, file exchange, read-only browsing).

**Outcome:** Agreed feature list aligned with teaching goals (e.g. publishing design topics, tracking each group’s topic and deliverables, storing learning materials).

---

## Phase B — Conceptual design (E-R model)

**Goal:** Express entities and relationships independent of SQL.

**What you produce:** An **E-R diagram** (e.g. PowerDesigner or equivalent) linking **students**, **teachers**, **question bank entries**, **per-topic student project records**, and **learning resources**.

**Outcome:** A model that can be checked for redundancy and integrity before coding tables.

---

## Phase C — Logical design (relational schema)

**Goal:** Map the E-R model to **relations**, keys, and referential dependencies.

**What you produce:** A **relational schema** such as:

- **Student** — student ID (primary key), password, name, class, chosen **topic ID** (foreign key to the question bank, nullable until a topic is chosen).
- **Teacher** — payroll ID (primary key), password, name.
- **Question bank** — topic ID, title/text of the problem.
- **Student project (per topic)** — one row per **topic** used as the unit of group work: topic ID, **group leader** name, **progress** text, **report** file path or URL, **numeric grade**; linked to the question bank (and, in the shipped SQL script, to leader name in the student table).
- **Materials** — topic-related **resource** name and **download URL** (or path) for learning materials.

**Outcome:** A schema that supports all use cases in Phase A (exact column sets may follow your instructor’s template; the repo includes a reference script `coursedesign.sql`).

---

## Phase D — Physical design (MySQL)

**Goal:** Implement the logical schema in **MySQL** with appropriate **primary keys**, **foreign keys**, and **indexes** on heavily queried columns (e.g. topic ID).

**What you produce:** Executable **DDL** (create tables, constraints, indexes) and optional seed data for demos.

**Outcome:** A database instance the application can connect to; storage and indexing choices briefly justified (e.g. B-tree indexes on keys and topic ID).

---

## Phase E — Application architecture (overview)

**Goal:** Split the system into **login** vs **post-login** behavior and separate **UIs** per role.

**What you produce:** **Software structure** description: e.g. login page with three entry modes (student / teacher / guest), then **student**, **teacher**, and **guest** operation pages as in the course’s UI specification.

**Outcome:** Clear navigation from authentication to role-specific features (the repo’s backend is REST + CORS; a front-end may live elsewhere).

---

## Feature area 1 — Student role

**Authentication**

- Log in with **student ID** and **password** (default password is typically the student ID until changed).
- **Change password** while authenticated.

**Topic selection**

- Choose **one** topic from the **question bank**.
- Register **group metadata** as required by the course (including **group leader** and, in the full specification, **teammates’ names, IDs, and class**). **After the choice is saved, it must not be changeable.**

**Queries**

- View **own topic selection** (e.g. name + problem title).
- View **course design grade** for the associated project record.

**Deliverables**

- **Upload** staged **work reports**; the system stores a **reference** (e.g. URL/path) on the group’s project row for that topic.

**Backend endpoints (representative):** student login, password update, query selection, query grade, choose topic (with leader handling), file upload for reports.

---

## Feature area 2 — Teacher role

**Authentication**

- Log in with **payroll ID** and **password** (default often equal to payroll ID until changed).
- **Change password**.

**Teaching operations**

- **Inspect** students’ topic choices (join of students and question bank).
- **Inspect** uploaded **reports** / project rows (full group table listing).
- **Set project progress** text for a given **topic** and **group leader** key.
- **Assign or update numeric grades** for that project row.

**Content management**

- **Add new topics** to the question bank (auto-generated or sequential topic IDs, depending on policy).
- **Upload learning materials**: store **resource name** and **download path/URL** in the materials table (often via multipart upload).

**Backend endpoints (representative):** teacher login, password update, list selections, list project rows, set progress, set grade, add topic, upload material.

---

## Feature area 3 — Guest (visitor) role

**Goal:** No account required.

**What guests can do**

- **Browse** the list of **course design topics** from the question bank.
- **Browse** **learning materials** (metadata and download links).

**Backend endpoints (representative):** public listing of topics and resources (same data teachers may also use when adding content).

---

## Phase F — Integration and deployment concerns

**Goal:** Make the system runnable end-to-end.

**What you address (at a high level):** application server configuration, **file upload** storage directory and URL generation for downloads, **database connection** settings, and **cross-origin** access if the UI is separate from the API.

**Outcome:** Students and teachers can complete the workflows above against a live database; guests can read public catalogs.

---

## Repository map (for orientation)

| Artifact | Role |
|----------|------|
| `README.md` | Chinese design document (requirements, E-R reference, UI sketches—**not** copied into this overview) |
| `coursedesign.sql` | Reference **MySQL schema** |
| `CourseDesignDBMS/…/controller/Handler.java` | Core **REST** operations for auth, topics, grades, listings |
| `CourseDesignDBMS/…/controller/UploadController.java` | **Multipart** uploads for student reports and teacher materials |
| `CourseDesignDBMS/…/mapper/DataBase.java` | MyBatis **data-access** layer to the Chinese-named tables |

---

## Summary

| Phase / area | You deliver |
|----------------|------------|
| A | Requirements for three roles |
| B | E-R model |
| C | Relational schema |
| D | MySQL DDL + indexes |
| E | Application structure + pages per role |
| Student features | Login, password, immutable topic choice, queries, report upload |
| Teacher features | Login, password, monitoring, progress & grades, topic & material upload |
| Guest features | Read-only topics and materials |
| F | Runnable integrated system |

---

*This overview is inferred from `README.md`, `coursedesign.sql`, and the Spring controllers only. Course staff may require extra fields (e.g. full teammate records) or security hardening beyond what this codebase shows.*
