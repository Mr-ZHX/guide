# Hotel Guest-Room Management System (`db-design`) — Experiment Overview

This repository is a **single integrated database-course project** (宾馆客房管理系统): a **MySQL** schema plus a **Spring Boot + MyBatis** REST backend. There are **no separate numbered lab folders** in the tree; the work is naturally split into **thematic deliverables** described below.

This document is for readers who know nothing about the assignment: **what must exist and what it should do**. It does **not** explain how to implement it and does **not** reproduce student reports.

**Reference in repo:** `README.md` (requirements sketch, core DDL, trigger intentions, stack).  
**API docs (when running):** `http://localhost:8081/doc.html` (Knife4j). Backend base URL: `http://localhost:8081`.

---

## Phase 1 — Requirements and relational design

**Goal:** Define a small **hotel / guest-room** domain: guests (**users**), **rooms**, and **stays** linking users to rooms.

**What you produce:** A clear **relational model** (tables, keys, roles). Ordinary guests vs **administrators** are distinguished (in this design, admins are not a separate table; a **role flag** on the user record is used).

**Functional expectations:** Support registration data such as nickname, legal name, phone, gender, ID card, password; rooms have number, type, in-room phone, **price per day**, and **occupancy status**; a stay links **user** and **room** and records timing (e.g. expected stay / duration fields as required by your spec).

---

## Phase 2 — Physical database (DDL) and indexing

**Goal:** Implement the schema in **MySQL** with appropriate types, **primary keys**, **timestamps**, and **logical delete** flags where specified.

**What you produce:** `CREATE TABLE` scripts (or migrations) for at least **`user`**, **`room`**, and **`user_room`**, consistent with the design document in `README.md`.

**Functional expectations:** Tables support the later features (roles, room status, soft delete). **Indexes:** primary keys on surrogate `id` columns for all tables (as described in the README).

**Extended schema (in code):** The implemented backend also persists **room consumables**, **per-guest consumption**, and **bills**—so the physical model goes beyond the three base tables once those features are included.

---

## Phase 3 — Triggers for room occupancy consistency

**Goal:** Keep **`room.status`** aligned with **check-in / check-out** events on **`user_room`**, using **database triggers** so the rule holds even if rows are inserted or logically updated outside one application path.

**What you produce:** Triggers (conceptually: **after** a new stay row, mark room occupied; **after** a stay ends or is logically removed, mark room free). Exact trigger names and events match your course specification.

**Functional expectations:** Opening a stay makes the room **occupied**; closing/leaving a stay returns the room to **vacant**, without relying only on application code.

---

## Phase 4 — Backend platform and persistence layer

**Goal:** Expose the database through a **REST API** using **Spring Boot** and **MyBatis** (XML mappers under `resources/mapper/`).

**What you produce:** A runnable service on port **8081**, unified JSON responses, global error handling, and **OpenAPI/Knife4j** documentation.

**Functional expectations:** CRUD and queries for all entities used by the UI flows are available via HTTP; parameters are validated and business failures return consistent error codes.

---

## Phase 5 — Authentication, session, and administrator authorization

**Goal:** **Register** and **login** users; store **session** state; restrict **admin-only** operations.

**What you produce:** Endpoints for registration and login; session attribute holding the logged-in user; an annotation-driven check (e.g. `@RoleCheck`) so only **role = administrator** can call management APIs.

**Functional expectations:** Guests can use stay and billing endpoints that require “self” context; they cannot perform admin actions. Admins can manage users, rooms, catalog items, and query bills as defined by the API.

---

## Phase 6 — Room catalog and administration

**Goal:** Manage the **inventory of rooms** and let clients **browse** them.

**What you produce (representative capabilities from `RoomController`):** Add/update/delete rooms (admin); list rooms; look up a room by room number.

**Functional expectations:** Room list supports filtering the UI by **room type** and **status**; only authorized staff alter the catalog.

---

## Phase 7 — Check-in and check-out (stay lifecycle)

**Goal:** A logged-in guest can **open** a stay in an available room and **check out**, updating **`user_room`** and (via triggers) **`room.status`**.

**What you produce:** APIs under `/userRoom` for **openRoom** and **outRoom**, enforcing that the **user id** in the request matches the session user.

**Functional expectations:** Prevents inconsistent stays (e.g. double booking depends on service rules); checkout path completes the stay and frees the room according to the database rules.

---

## Phase 8 — In-room consumables (catalog and guest usage)

**Goal:** Model **items** available in or tied to rooms (minibar, services, etc.); guests **consume** items during a stay.

**What you produce:** Admin CRUD for **room items**; search by name or room; listing APIs for items. Persistence for **per-user consumption** of items (`user_item` and related mappers/services).

**Functional expectations:** Items have prices usable in billing; consumption is attributable to the correct guest and stay context.

---

## Phase 9 — Billing, pricing, and checkout charges

**Goal:** Combine **room rate × duration** with **item consumption** into **bills**; support **saving** and **settling** bills; expose **consumption breakdowns** where appropriate.

**What you produce:** Bill APIs: persist a bill, settle (“out”) a bill, admin list/query bills, **calculate** total price from a request, fetch **item consumption detail** for a stay/room, fetch **room fee summary**.

**Functional expectations:** Totals reflect business rules encoded in `BillService`; users may only inspect **their own** consumption data (authorization checks on room/user linkage).

---

## Phase 10 — Front-end client (documented, may live outside this repo)

**Goal (per `README.md`):** A **Vue 3 + Element Plus** SPA: login/register, room listing with check-in/check-out actions, **admin** dashboards for users and rooms, and UX for **fees** (including consumables).

**Note:** This checkout of `db-design` contains the **backend** only; the Vue app may be in another directory or repository. The experiment still “includes” front-end work if your course requires end-to-end delivery.

---

## Summary table

| Phase | Focus | Main outcome |
|------|--------|----------------|
| 1 | Analysis & design | Relational model for users, rooms, stays |
| 2 | DDL | MySQL tables, keys, soft delete, indexes |
| 3 | Triggers | Room occupancy synced with stay rows |
| 4 | Stack | Spring Boot + MyBatis REST + API docs |
| 5 | Security | Register/login, session, admin gate |
| 6 | Rooms | Admin CRUD + public listing/search |
| 7 | Stays | Check-in / check-out APIs |
| 8 | Items | Catalog + guest consumption records |
| 9 | Bills | Calculate, save, settle, reports |
| 10 | UI | Vue client (if required by the course) |

---

*Derived from `README.md` and the Java API surface only; official grading rubrics may add constraints not listed here.*
