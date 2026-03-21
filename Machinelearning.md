# CS7641-ML — Assignment Framework Overview (English)

This repository is a **Fall 2020–style** implementation bundle for **Georgia Tech CS 7641 — Machine Learning** (OMSCS). The tree is split into **four top-level assignment folders** matching the course’s major projects.

This document is organized **only by those folders** (`A1-SL`, `A2-RO`, `A3-USL`, `A4-MDPs`). It states **what each part is for** and **what functionality the code is built to exercise**. It does **not** explain algorithms or how to code them. It does **not** reproduce written reports.

**Integrity:** If you are **currently enrolled** in CS 7641/4641, follow **Georgia Tech’s Honor Code** and your term’s collaboration rules. This file describes **repository layout**, not permission to submit someone else’s work.

---

## A1-SL — Supervised learning (`A1-SL/`)

**Goal:** Compare **supervised classifiers** on **two binary classification datasets** with different label balance (e.g. **Mobile Prices** and **Chess**, as named in the local `README`).

**Algorithms covered in the framework:**

- **Decision trees** (with pruning-related choices described in the handout text)
- **k-NN** (multiple **k** values)
- **Boosted trees** (boosted decision trees, e.g. AdaBoost-style setup)
- **SVMs** with **multiple kernel** options
- **Neural networks** (configurable depth/activations via scikit-learn)

**What the code is set up to do:** Load CSV data from **`Data/`** (handle categoricals via one-hot encoding, optional scaling), run **cross-validated hyperparameter search** (full or shortened grid), optionally emit **validation curves** and **learning curves**, and save figures under **`Images/`** and parameter logs under **`ParamTests/`**. Entry point: **`main.py`** → **`run_sl_algos(...)`** with flags to enable/disable each learner and optional fixed hyperparameters.

**Outcome:** Side-by-side empirical comparison of the five families on both datasets.

---

## A2-RO — Randomized optimization (`A2-RO/`)

**Goal:** Study **stochastic / population-based optimizers** on **toy fitness problems**, then use them to **train the Assignment 1 neural network** as an alternative to ordinary gradient descent.

**Algorithms in the framework (via `mlrose-hiive` + utilities):**

- **Randomized Hill Climbing (RHC)**
- **Simulated Annealing (SA)**
- **Genetic Algorithm (GA)**
- **MIMIC**

**Optimization domains (from `README` / `main.py` intent):** e.g. **Continuous Peaks**, **Traveling Salesperson**, **Knapsack** — each treated as a **maximization** problem with curves plotted vs **iterations**, **evaluations**, and **time**.

**Neural-network comparison:** Scripts such as **`NN_RHC.py`**, **`NN_SA.py`**, **`NN_GA.py`**, **`NN_GD.py`** search hyperparameters for each optimizer vs **SGD**, write CSV results, then **`NN_Graph.py`** (with best HPs filled in) produces **loss** and **ROC vs iteration** comparisons. Additional **`*-HPPlots.py`** files generate validation-style plots per NN+optimizer pair.

**Outcome:** Evidence of how each RO method behaves on standard problems and how it performs as a **NN weight trainer** on the Mobile Prices–style task.

---

## A3-USL — Unsupervised learning & dimensionality reduction (`A3-USL/`)

**Goal:** Run **clustering** and **dimensionality reduction (DR)** on the **same two datasets** as A1, then combine DR + clustering, and finally plug transformed data back into the **A1 neural network** pipeline.

**Clustering (scikit-learn):**

- **k-means**
- **Expectation–Maximization** (Gaussian mixture–style)

**Metrics:** **Unsupervised** scores (e.g. BIC, inertia, Davies–Bouldin, Calinski–Harabasz, silhouette) plus **ground-truth** agreement when labels exist (e.g. V-measure, adjusted Rand, adjusted mutual information).

**Dimensionality reduction:**

- **PCA** (explained variance ratio)
- **ICA** (kurtosis-oriented view in the README)
- **Random projections** (reconstruction error)
- **Locally Linear Embedding (LLE)** (reconstruction error)

**Pipeline behavior:** After standalone clustering/DR studies, rerun clustering on **DR-transformed** data; then, for one dataset, train the NN on **reduced features** and on **features augmented with cluster assignments**, comparing ROC-style behavior. Entry: **`main.py`** → **`run_ul_algos(...)`**.

**Outcome:** Linked analysis of clustering quality, DR choice, and impact on downstream classification.

---

## A4-MDPs — Reinforcement learning on MDPs (`A4-MDPs/`)

**Note:** The folder’s `README.md` heading mistakenly says “Assignment 3”; the directory name and root index treat this as the **RL / MDP** assignment (**Assignment 4** in the root `ReadMe.md`).

**Goal:** Implement or run **tabular RL** on **two environments** at **small and large** sizes to compare scaling behavior.

**Algorithms (six driver scripts, paired by env + method):**

- **Value Iteration** — `VI-Forest.py`, `VI-FrozenLake.py`
- **Policy Iteration** — `PI-Forest.py`, `PI-FrozenLake.py`
- **Q-Learning** — `QL-Forest.py`, `QL-FrozenLake.py`

**Environments:**

- **Forest** management MDP (`mdptoolbox` example — default sizes **10** and **1000** states per README)
- **OpenAI Gym FrozenLake** — grid sizes **4** and **16** mentioned in README

**What the scripts produce (per README):** For VI/PI — **heatmaps** over **ε–γ** (or similar grids) for average reward, iterations to convergence, runtime; curves of **value / error / reward vs iteration**; **policy** visualizations. For Q-learning — logged parameters/rewards for external plotting, value/error/reward curves, policy plots. Figures go under **`Images/`**.

**Dependencies:** `gym`, `mdptoolbox-hiive`, `mdptoolbox` (as listed locally).

**Outcome:** Empirical comparison of the three methods across **small vs large** MDPs.

---

## Summary table (folders in this repo)

| Folder | Topic |
|--------|--------|
| `A1-SL/` | Five supervised learners × two datasets, CV + curves |
| `A2-RO/` | RHC, SA, GA, MIMIC on toy problems + NN training vs SGD |
| `A3-USL/` | Clustering + DR + combined pipelines + NN on transformed data |
| `A4-MDPs/` | VI, PI, Q-learning on Forest + FrozenLake |

---

*Official rubrics, deadlines, and allowed libraries change by term; use the current CS 7641 course page as authority.*
