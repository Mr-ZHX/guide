# UvA Deep Learning 1 (2022) — Assignment Framework Overview (English)

This repository holds the **Fall 2022** practical work for **Deep Learning 1** at the **University of Amsterdam (UvA)** (*UvA Deep Learning Course*, [course site](https://uvadlc.github.io/)). The tree is organized as **`assignment1/`**, **`assignment2/part1`**, **`assignment2/part2`**, **`assignment3/part1`**, **`assignment3/part2`**, plus **Conda** environment files (`dl2022_*.yml`).

This document follows **only those folders**. It states **what each assignment is for** and **what the code is meant to do or produce**. It does **not** teach theory or fill-in-the-blank solutions. It does **not** copy student reports or reuse **SLURM log** contents as specifications.

---

## Assignment 1 — MLPs and backpropagation (`assignment1/`)

**Goal:** Work through **backpropagation** for basic building blocks of a feed-forward net, implement them, and train a **multi-layer perceptron (MLP)** on **CIFAR-10**.

**Framework pieces:**

- **`modules.py` / `mlp_numpy.py` / `mlp_pytorch.py`** — modular layers and full MLPs (NumPy and PyTorch tracks).
- **`train_mlp_numpy.py` / `train_mlp_pytorch.py`** — training entry points.
- **`cifar10_utils.py`** — data handling for CIFAR-10.
- **`unittests.py`** — correctness checks for implemented modules.
- **`compare_models.py`** — compare runs (accuracies, hyperparameters, statistics).
- **`confusion_matrix.py`** — evaluation visualization support.

**Outcome:** A trained MLP on CIFAR-10 with **documented** accuracy and behavior, consistent with the course’s comparison and reporting expectations.

---

## Assignment 2 — CNN transfer learning & visual prompting (`assignment2/`)

Split into **two parts** (the course text also mentions **GNN** reading without coding obligations).

### Part 1 — Transfer learning (`assignment2/part1/`)

**Goal:** Practice **ImageNet-pretrained CNNs** and adapt them to **CIFAR-100**.

**What you do (from `assignment2/README.md`):**

- Survey **PyTorch torchvision** classification models / weights (ImageNet).
- **Benchmark** chosen architectures: **inference speed**, **memory**, **parameter count**.
- **Fine-tune ResNet-18** for CIFAR-100 by replacing the **final classifier** head.
- Use **data augmentation** to improve accuracy.

**Framework:** **`transfer_learning.py`**, **`train.py`**, **`cifar100_utils.py`**, notebook **`CNN-Transfer-Learning.ipynb`**, SLURM-style **`.job`** scripts.

**Outcome:** Tables/plots comparing models plus a **fine-tuned** ResNet-18 with augmentation experiments.

### Part 2 — CLIP visual prompting (`assignment2/part2/`)

**Goal:** Use **CLIP (ViT-B/32)** for **image classification** without full fine-tuning of the backbone: **zero-shot**, **hand-crafted text prompts**, and **learned visual prompts**; then study **robustness** and **cross-dataset** behavior.

**Deliverables (from READMEs):**

- **Zero-shot:** complete/evaluate **`clipzs.py`** on **CIFAR-10** and **CIFAR-100**.
- **Custom tasks:** vary **`--prompt`** and **`--class_names`**, optional **`--visualize_predictions`**.
- **Learned prompts:** train with **`learner.py`**, **`vpt_model.py`**, **`vp.py`**.
- **Adversarial / stress tests:** e.g. prompt design driving **near-random** CIFAR-100 accuracy (course question).
- **Robustness:** **`robustness.py`** under distribution shift / noise settings.
- **Cross-dataset:** **`cross_dataset.py`** — apply prompts trained on one split to the **concatenation** of CIFAR-10 and CIFAR-100.

**Supporting code:** **`dataset.py`**, **`main.py`**, **`utils.py`**, etc.

**Outcome:** Reported accuracies, visualizations, and analysis files under **`results/`** (logs are run artifacts, not the spec itself).

---

## Assignment 3 — Deep generative models (`assignment3/`)

Two **coding** parts (folder README title also mentions “3D vision” reading; the **implemented** track here is **VAE + AAE**).

### Part 1 — Variational autoencoder (`assignment3/part1/`)

**Goal:** Implement a **VAE** in **PyTorch Lightning** (`train_pl.py`) for **generative modeling** of **discrete (4-bit) MNIST**: pixels quantized to **16** gray levels.

**Framework:** **`cnn_encoder_decoder.py`** (encoder/decoder templates), **`mnist.py`** (discretized MNIST loaders), **`utils.py`** (ELBO-related utilities, sampling, **bits-per-dimension**, latent **manifold** visualization), **`unittests.py`**.

**Functional targets:** Training that minimizes **reconstruction + KL**-style objective, reporting **BPD**, **sampling** new digits, **2D/20D latent** runs as per handout, passing **unit tests**.

**Outcome:** Checkpointed VAE that generates plausible **4-bit MNIST** samples and meets metric/visualization requirements.

### Part 2 — Adversarial autoencoder (`assignment3/part2/`)

**Goal:** Implement an **AAE** (encoder + decoder + **discriminator** on latent codes) on **standard MNIST** in **`models.py`**, with training orchestration in **`train.py`**.

**Functional targets:** Correct **adversarial** and **reconstruction** losses, alternating **optimizer** steps, logged samples per epoch, passing **`unittests.py`** for sub-networks.

**Outcome:** Trained AAE that generates **MNIST**-like digits under default hyperparameters in reasonable GPU time.

---

## Summary table (repository layout)

| Path | Topic |
|------|--------|
| `assignment1/` | NumPy + PyTorch MLP on CIFAR-10 |
| `assignment2/part1/` | Pretrained CNN metrics + ResNet-18 on CIFAR-100 |
| `assignment2/part2/` | CLIP zero-shot, prompt engineering, VPT, robustness, cross-dataset |
| `assignment3/part1/` | VAE on 4-bit MNIST (PyTorch Lightning) |
| `assignment3/part2/` | AAE on MNIST |

---

*Question numbers (e.g. Q1.7) refer to the official PDF for that term. For submission, exclude datasets and trained checkpoints as instructed in `assignment3/README.md`.*
