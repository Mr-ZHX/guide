# USTC Computer Graphics (2024 Spring) — Lab Framework Overview

This document summarizes **only what is present in this repository** under `USTC_CG_24/`: numbered homework folders in `Homeworks/`, shared frameworks `Framework2D/` and `Framework3D/`, plus course meta files (`README.md`, `Softwares/`). It is written for readers **unfamiliar** with the course: **what each lab is**, **what you are expected to produce**, and **where code lives**. It **does not** explain implementation steps or reproduce student-style lab reports.

**Primary in-tree sources:** `README.md`, `Homeworks/README.md`, and each `Homeworks/<id>_<name>/README.md`.

---

## Repository layout (labs vs. frameworks)

| Area | Role |
|------|------|
| `Homeworks/0_cpp_warmup/` | Standalone CMake C++ warmup (`project/`, `documents/`, `samples/`) |
| `Homeworks/1_mini_draw/` … `2_image_warping/` … `3_poisson_image_editing/` … `4_tutte_parametrzation/` … `5_arap_parametrzation/` | Assignments whose **implementation target** is the shared **`Framework2D/`** tree (per each homework README) |
| `Homeworks/6_shaders/` … `7_path_tracing/` … `8_mass_spring/` … `9_sph_fluid/` … `10_character_animation/` | Assignments whose **implementation target** is the shared **`Framework3D/`** tree |
| `Framework2D/` | 2D/GUI + image + mesh-node coursework framework |
| `Framework3D/` | 3D rendering, simulation, and path tracing coursework framework |
| `Softwares/` | Tooling notes (Git, CMake, etc.); not a programming lab |

---

## Lab 0 — C++ warmup (`Homeworks/0_cpp_warmup/`)

**Goal:** Build comfort with **CMake**, **Visual Studio** workflow, and core **C++ / OOP** mechanics before the graphics sequence.

**What you do:** Complete **five sequential mini-exercises** in `project/`, each documented under `documents/`:

1. **Basic dynamic array** — transition from C-style coding toward C++ classes; constructors, destructors, overloading; pointers and dynamic allocation.  
2. **Efficient dynamic array** — deepen memory/pointer practice; emphasize clean **public interfaces**.  
3. **Templated dynamic array** — C++ **templates**; relate to **`std::vector`** semantics.  
4. **Polynomial with `std::list`** — **dynamic libraries (DLL)** usage; STL **`list`**.  
5. **Polynomial with `std::map`** — **static libraries**; STL **`map`**.

**Deliverables:** Working code for the warmup project; submission policy is **code-focused** in the homework README (course also expects reports for later labs). Reference solutions exist under `samples/` for comparison **after** attempting each part.

---

## Lab 1 — MiniDraw (`Homeworks/1_mini_draw/` + `Framework2D/`)

**Goal:** Interactive **2D drawing** with a small GUI; practice **OOP** (encapsulation, inheritance, polymorphism).

**What you implement:** A **MiniDraw** program that can create and display primitive shapes: at minimum **line, ellipse, rectangle, polygon**; each primitive type as its own class; all primitives derive from a common base (e.g. **`Shape`**); drawing invoked through **polymorphism**.

**Optional extensions (from README):** freehand mode, line width/color, fill, selection, transforms, vertex editing, etc.

**Deliverables:** Code in **`Framework2D/`** (as directed by the homework README) plus a **lab report**.

---

## Lab 2 — Image warping (`Homeworks/2_image_warping/` + `Framework2D/`)

**Goal:** Implement **image deformation** driven by control points using at least two classical scattered-data interpolation schemes, with clean separation between **mathematical warp** and **image sampling**.

**What you implement (required):**

- **IDW** (inverse distance-weighted interpolation), per course documentation.  
- **RBF** (radial basis function interpolation), per course documentation.

**Optional:** hole / seam filling (e.g. using **ANN** as referenced in documents).

**Supporting skills:** Solving linear systems (hand-written or via **Eigen**; example CMake projects exist under `documents/`).

**Deliverables:** Code in **`Framework2D/`**, report; maintain **decoupled** warp abstraction + OOP structure as stated in README.

---

## Lab 3 — Poisson image editing (`Homeworks/3_poisson_image_editing/` + `Framework2D/`)

**Goal:** **Seamless cloning** / gradient-domain fusion (Poisson image editing) with interactive feedback.

**What you implement (required):**

- Poisson editing with a **rectangular** boundary region.  
- At least the **seamless cloning** application described in the handout.  
- **Real-time** preview while dragging the pasted region.

**Optional:** **General (polygonal) boundaries** via polygon rasterization / scan-line ideas documented in `documents/`.

**Supporting skills:** Large **sparse** linear systems with **Eigen**, including **pre-factorization** for efficiency.

**Deliverables:** Code in **`Framework2D/`**, report.

---

## Lab 4 — Tutte mesh parameterization (`Homeworks/4_tutte_parametrzation/` + `Framework2D/`)

**Goal:** Compute **UV parameterizations** of triangle meshes using **Tutte-type** methods (Floater 1997 family), visualize with textures, and work inside the course **node graph** workflow.

**What you implement (required):**

- **Minimal surface** with **fixed boundary** via a sparse linear solve.  
- **Parameterization** by mapping the mesh boundary to a **planar convex polygon** and solving for interior coordinates.  
- Experiment with **multiple weight schemes** (at least **uniform** and **cotangent**; **Floater shape-preserving** optional).  
- Validate on provided **meshes and textures**.

**Deliverables:** Report; packaged submission including **`Blueprints.json`**, `nodes/`, optional `utils/`, and `data/` as specified in the homework README.

---

## Lab 5 — ARAP mesh parameterization (`Homeworks/5_arap_parametrzation/` + `Framework2D/`)

**Goal:** Implement **as-rigid-as-possible (ARAP)** surface parameterization from the cited paper, compare behavior to Tutte (Lab 4), and handle **local/global** optimization structure.

**What you implement (required):**

- Full **ARAP** parameterization pipeline in the node/framework setting.  
- Use test meshes/textures for qualitative verification.

**Optional:** **ASAP** and **hybrid** variants from the same literature line.

**Supporting skills:** Nonlinear iteration, **SVD** of small matrices, sparse linear solves, mesh programming.

**Deliverables:** Same packaging pattern as Lab 4 (**`Blueprints.json`**, nodes, utils, data, report).

---

## Lab 6 — Shaders (`Homeworks/6_shaders/` + `Framework3D/`)

**Goal:** Real-time **OpenGL + GLSL** rendering with standard material lighting and shadows.

**What you implement (required):**

- **Blinn–Phong** shading.  
- **Shadow mapping** (depth pass + shadow test in lighting).

**Optional:** **PCSS**-style soft shadows; **SSAO** (screen-space ambient occlusion).

**Provided context:** Baseline rasterization pass and test scenes (external data link in README).

**Deliverables:** Report; submit **`shaders/`**, **`nodes/`**, optional **`utils/`**, and graph JSON files (**`CompositionGraph.json`**, **`GeoNodeSystem.json`**, **`RenderGraph.json`**) per README.

---

## Lab 7 — Path tracing (`Homeworks/7_path_tracing/` + `Framework3D/`)

**Goal:** Monte Carlo **physically based** rendering in the course’s **`hd_USTC_CG`** renderer subtree.

**What you implement (required):**

- **Direct lighting** integrator with **importance sampling** for **rectangular area lights**.  
- **Path tracing** with **Russian roulette** termination.

**Optional:** Additional **material** with importance sampling + **MIS** comparisons; **transparent** materials.

**Deliverables:** Entire modified **`hd_USTC_CG/`** tree as instructed, plus report (and external test scenes per README).

---

## Lab 8 — Mass–spring simulation (`Homeworks/8_mass_spring/` + `Framework3D/`)

**Goal:** Simulate **mass–spring systems** on meshes with stable time integration.

**What you implement (required):**

- **Semi-implicit** and **implicit** integrators for mass–spring dynamics (Part 1 documentation).

**Optional:** Penalty-based **cloth–sphere collision**; **fast mass–spring** via local/global iteration (Liu et al. 2013, Part 2 documentation).

**Deliverables:** Files under `Framework3D/.../geometry/mass_spring/` and `node_mass_spring.cpp` (plus graph JSONs if applicable), report—see homework README for exact bundle layout.

---

## Lab 9 — WCSPH fluid (`Homeworks/9_sph_fluid/` + `Framework3D/`)

**Goal:** **Particle-based fluid** using **weakly compressible SPH (WCSPH)**.

**What you implement (required):**

- Full WCSPH loop: **density estimation**, **viscosity**, **pressure**, **velocity/position updates** (Part 1 documentation).

**Optional:** Surface reconstruction from particles + **path-traced** rendering of the surface; **IISPH** incompressibility (Part 2 documentation).

**Deliverables:** Files under `Framework3D/.../geometry/sph_fluid/` and `node_sph_fluid.cpp`, graph JSONs, report.

---

## Lab 10 — Character animation (`Homeworks/10_character_animation/` + `Framework3D/`)

**Goal:** **Skeleton-driven** deformation of a skinned mesh.

**What you implement (required):**

- Per-bone **joint transforms** and **skinning** (vertex blending) updates.

**Optional:** **Cloth** on the character using Lab 8-style springs with simple **body–cloth collision** (Part 2 documentation).

**Deliverables:** Files under `Framework3D/.../geometry/character_animation/` and `node_animate_mesh.cpp`, graph JSONs, report.

---

## What this overview deliberately omits

- **Step-by-step algorithms** inside `documents/*_step_by_step.md` and similar tutorials.  
- **Grading rubrics**, due-date enforcement, and CES submission UI details (see `Homeworks/README.md`).  
- **External download links** for datasets—only noted where the homework README states they exist.

For build environment and Git workflow, see `Softwares/` and the root `README.md`.
