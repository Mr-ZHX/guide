# University of Edinburgh MLP — Lab & Coursework Framework Overview (English)

This document summarizes **what is actually present** in this `mlp/` repository: the **`mlp` Python package**, **Jupyter notebooks**, **coursework specifications** (`spec/`), **report templates** (`report/`), **helper scripts** (`scripts/`), and **notes** (`notes/`). It is meant for someone unfamiliar with the course: **what each lab-style unit is about** and **what you are expected to produce**, without implementation recipes or student-style reports.

**Source of truth in-tree:** `README.md`, notebooks under `notebooks/`, `spec/coursework1.tex`, `spec/coursework2.tex`, and the `mlp/*.py` modules.

---

## Repository role (from `README.md`)

**Machine Learning Practical (MLP)** is an assignment-driven Informatics course focused on **building, training, and evaluating** small neural systems in Python/NumPy. Students **implement and extend** parts of the **`mlp`** package while working through **Jupyter lab notebooks**.

---

## Shared code framework — `mlp/` package

The installable package (see `setup.py`) provides the course’s NumPy-based neural-network building blocks, including (by module name):

- **`layers`** — layer types (affine transforms, nonlinearities, etc.; coursework adds or completes more).
- **`models`** — composition of layers into trainable models.
- **`errors`** — loss / objective interfaces (e.g. softmax + cross-entropy for classification).
- **`learning_rules`** — parameter update rules (extended in coursework, e.g. Adam, RMSProp).
- **`optimisers`** — training loop driver around models, data, and learning rules.
- **`data_providers`** — batched data loading (MNIST-style; EMNIST in CW2 per spec).
- **`initialisers`**, **`schedulers`** — weight init and learning-rate schedules.

**What you do in general:** fill in or extend these modules as directed by notebooks and coursework PDFs, then validate behavior with notebook exercises, unit-test notebooks, or generated submission files.

---

## In-repo lab sequence — `notebooks/01_…` through `07_…`

These notebooks are the **weekly lab track** (tutorial + coding exercises). Titles and aims below come from the notebook intros.

| Notebook | Topic (high level) | What you work on |
|----------|-------------------|------------------|
| **`01_Introduction.ipynb`** | Jupyter workflow | Using the notebook server, cells, kernels, Markdown; baseline Python/NumPy setup for the course. |
| **`02_Single_layer_models.ipynb`** | Single-layer models | Affine maps, forward propagation, gradients, and gradient-based training for a one-layer model; hyperparameter exploration. |
| **`03_Multiple_layer_models.ipynb`** | Multi-layer stacks | Stacking **affine** and **nonlinearity** layers, `MultipleLayerModel`, training deeper nets (foundation for later coursework). |
| **`04_Generalisation_and_overfitting.ipynb`** | Generalisation | Overfitting vs. generalisation, train/validation behavior, complexity in a simple regression setting. |
| **`05_Non-linearities_and_regularisation.ipynb`** | Regularisation | Penalties (e.g. L1/L2-style complexity terms) to control overfitting; ties to lecture material on hidden layers / regularisation. |
| **`06_Dropout_and_maxout.ipynb`** | Dropout & maxout | Stochastic regularisation (**dropout**) and **maxout**-style piecewise units; training patterns on MNIST-like classification. |
| **`07_Autoencoders.ipynb`** | Autoencoders | Unsupervised / reconstruction-style networks built from the same layer toolkit. |

Supporting assets: `notebooks/res/` (figures, diagrams).

---

## Coursework specifications — `spec/`

Formal write-ups (LaTeX source) describe **graded assignments**. Your clone includes **`coursework1.tex`** and **`coursework2.tex`** (with bibliographies). Below is a **functional summary** only.

### Coursework 1 (MNIST deep nets & activations)

**Theme:** Train **multi-layer** networks for **MNIST** digit classification and study **hidden activation** variants related to ReLU.

**Standard experiment setup (fixed for comparability):** Two hidden layers, **100** units each, softmax output; shared training protocol (batch size **50**, **100** epochs for reported runs, etc.—see spec).

**Part 1 — New activation layers (code):**

- Implement **`LeakyReluLayer`**, **`EluLayer`**, and **`SeluLayer`** with forward (`fprop`) and backward (`bprop`) methods.
- Verify with a dedicated tests notebook (**`Activation_Tests.ipynb`**) and generate a **personalised numeric output file** using a **`generate_inputs.py`**-style script, as described in the spec.

**Note:** In **this** workspace snapshot, `Activation_Tests.ipynb` and `generate_inputs.py` are **not** present under `notebooks/` or `scripts/`; the coursework text still describes them as part of the full assignment bundle (often on a separate git branch in the upstream course repo).

**Part 2 — Experiments & report:**

- **2A:** Compare Leaky ReLU, ELU, and SELU against **sigmoid** and **ReLU** baselines (same width/depth).
- **2B:** Deeper networks (**2–8** hidden layers), fixed width; study **initialisation** schemes (fan-in / fan-out / Glorot-style uniform variants; optional Gaussian variants for SELU as cited in spec).
- Submit a **short ICML-style PDF report** using templates in `report/` (`mlp-cw1-template.tex` / style files).

### Coursework 2 (EMNIST, optimisers, batch norm, convolutions)

**Theme:** **EMNIST Balanced (47 classes)** image classification—deeper nets, **adaptive optimisers**, **batch normalisation**, and **convolutional** models.

**Part A — Deep neural networks on EMNIST:**

- Establish **baseline** DNNs (architecture, activations, regularisation, dropout, etc.—students choose a focused subset).
- Implement **`RMSProp`** and **`Adam`** as new classes derived from **`GradientDescendLearningRule`** in `mlp/learning_rules.py`.
- Compare **SGD vs. RMSProp vs. Adam** on EMNIST.
- Implement **`BatchNormalizationLayer`** (`fprop`, `bprop`, `grads_wrt_params`).
- Validate with **`BatchNormalizationLayer_tests.ipynb`**.
- Generate **`sXXXXXXX_batchnorm_test.txt`** via **`scripts/generate_batchnorm_test.py`** (student-ID-seeded vectors).

**Part B — Convolutional networks:**

- Implement **`ConvolutionalLayer`** (`fprop`, `bprop`, `grads_wrt_params`); validate with **`ConvolutionalLayer_tests.ipynb`**.
- Generate **`sXXXXXXX_conv_test.txt`** via **`scripts/generate_conv_test.py`**.
- Implement **max-pooling** (non-overlapping required; strided/general version optional).
- Train and evaluate **1- and 2-convolution** architectures on EMNIST with default kernel/pool sizes given in the spec; report results.

**Framework additions described in spec:** `ReshapeLayer`, **`EMNISTDataProvider`**, bundled EMNIST splits (on full coursework branch upstream).

**Starter experiments:** `notebooks/Coursework_2.ipynb` provides plotting/training helpers; full instructions in `spec/coursework2.tex`.

**Report:** `report/mlp-cw2-template.tex` (page limit and format per spec).

---

## Verification & submission artifacts (in-tree)

| Artifact | Purpose |
|----------|---------|
| `scripts/generate_batchnorm_test.py` | Produce CW2 batch-norm autograder-style output file. |
| `scripts/generate_conv_test.py` | Produce CW2 convolution autograder-style output file. |
| `notebooks/BatchNormalizationLayer_tests.ipynb` | Unit checks for batch norm layer API. |
| `notebooks/ConvolutionalLayer_tests.ipynb` | Unit checks for convolution layer API. |
| `report/*.tex`, `*.sty`, `*.bst` | LaTeX templates and bibliography style for PDF reports. |

---

## Notes & environment — `notes/`

Operational documentation (e.g. environment setup, lab machines, remote notebooks). Not programming tasks themselves.

---

## Data — `data/`

Contains at least one bundled text dataset file (e.g. weather-related series). MNIST/EMNIST paths are typically configured via **`MLP_DATA_DIR`** (see `README.md` / setup notes); full EMNIST files may be obtained per course instructions.

---

## What this overview omits

- **Equations and citation details** inside the `.tex` sources (they duplicate the PDF handouts).
- **Step-by-step lab solutions** and **notebook exercise answers**.
- **University submission mechanics, deadlines, and marking** (except brief pointers that reports/tests exist).

If your teaching team uses additional branches (e.g. `mlp2017-8/coursework1`) for missing `Activation_Tests.ipynb` / `generate_inputs.py`, sync that branch to match the spec’s file list.
