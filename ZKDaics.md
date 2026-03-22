# 2024-AICS-EXP — Lab Overview (English)

This document summarizes **what each lab-like unit in this repository is about**, based on **directory names, code layout, and official `实验题目.md` text where present** under the **`实验答案` (solution) trees**. It is written for readers who are **not** familiar with the course. It does **not** explain implementation steps and does **not** reproduce student lab reports.

**Scope of sources**

- **`实验答案/`** — main numbered labs (二, 三, 四, 五三选一, 六, 七, 八) with reference code.
- **`开发与实践/实验答案/`** — separate “development & practice” track (**YOLOv5**, **Swin Transformer**, **BERT**).

The top-level **`实验题目/`** files for 实验二–五 are mostly **cloud-disk links** in this snapshot; where detailed objectives exist (e.g. 开发与实践, 实验八 variants), they are paraphrased at a high level.

---

## What is *not* in the solution tree

- There is **no** `实验答案/实验一` folder in this repository snapshot, so **Lab 1** is not represented in the answer code here.

---

## Main track — `实验答案/`

### Lab 2 (`实验二/`)

**Sub-projects:** `exp_2_1`, `exp_2_2` (each with `stu_upload/` Python).

**Artifacts:** `mnist_mlp_cpu.py`, `layers_1.py`, `mnist_mlp_demo.py` (and parallel structure across sub-experiments).

**What it is about (inferred):** introductory **MNIST** classification with a **multi-layer perceptron**—building or extending **layers** and running training/inference on **CPU** (and related demo scripts). Exact grading and step-by-step tasks are not in the local `实验题目.md` (link-only).

---

### Lab 3 (`实验三/`)

**Sub-projects:** `exp_3_1`, `exp_3_2`, `exp_3_3`.

**Artifacts:** e.g. `vgg_cpu.py`, `vgg19_demo.py`, `exp_3_3_style_transfer.py`, modular `layers_*.py`.

**What it is about (inferred):** **CNN-style** models (VGG-related demos) and a **neural style transfer** exercise using composable layer modules. The third sub-experiment clearly targets **style transfer** behavior, not only classification.

---

### Lab 4 (`实验四/`)

**Sub-projects:** `exp_4_1` … `exp_4_4`.

**Artifacts (high level):**

- `exp_4_1`: `generate_pth.py`, `evaluate_cpu.py`, `evaluate_cnnl_mfus.py` — model artifact generation and **CPU vs CNNL/Mfus** evaluation.
- `exp_4_2`: evaluation scripts for **CPU** and **CNNL Mfus** paths.
- `exp_4_3`: `train.py`, `train-mlu.py` — **training** on CPU and MLU-oriented training.
- `exp_4_4`: custom **`hsigmoid`** in C++ (`hsigmoid.cpp`), Python tests (`test_hsigmoid.py`), `setup.py` — **custom operator / extension** packaging and correctness checks.

**What it is about (inferred):** a progression from **export/evaluate** workflows, through **training**, to implementing and testing a **custom activation** as a compiled extension with evaluation harnesses.

---

### Lab 5 — pick one (`实验五三选一实验/`)

**Tracks in the answers (three options):**

| Subfolder | Purpose (from filenames) |
|-----------|----------------------------|
| `yolov5_train/` | YOLOv5-oriented **training** utilities (`general.py`, `common.py`, `augmentations.py`). |
| `tacotron2_inference/` | **Tacotron2**-style **inference** (`model.py`, `test_infer.py`). |
| `bert_train/` | **BERT** on **SQuAD**-style training entry (`run_squad.py`). |

**What it is about:** students choose **one** of three application areas—**object detection (YOLOv5)**, **speech/TTS (Tacotron2)**, or **QA/BERT (SQuAD)**—and complete the corresponding pipeline. Local `实验题目.md` is link-only; details are assumed to live in the course handout.

---

### Lab 6 (`实验六/`)

**Artifacts:** Verilog sources under `src/` — **`serial_pe`**, **`parallel_pe`**, **`matrix_pe`** (`serial_pe.v`, `parallel_pe.v`, `matrix_pe.v`, `pe_mult.v`, `pe_acc.v`, …); **ModelSim** scripts (`sim/*.do`, `tb_top_*.v`, `compile_*.f`); `data/data_gen.py` and binary/text stimulus files under `data/`.

**What it is about:** **hardware description** of **processing elements (PEs)** for multiply/accumulate style workloads, evolving from **serial** to **parallel** to **matrix** organizations, with **simulation** testbenches and generated data. The included `README.md` only notes a practical fix for **simulation data generation** (not a full spec).

---

### Lab 7 (`实验七/`)

**Note:** internal folders are named `exp_5_1_*` and `exp_5_2_*` (legacy numbering); they sit under **`实验七`** in this tree.

**`exp_5_1_custom_pytorch_mlu_op/`**

- **Purpose:** integrate a **custom PyTorch extension** built with **MLUExtension**-style packaging: BangC/Bang sample kernel (e.g. **sigmoid**), C++ glue, Python module, and **tests** (`tests/test_sigmoid.py`, `test_sigmoid_benchmark.py`).
- **Outcomes:** installable custom op, correctness and **benchmark** hooks against CPU reference behavior.

**`exp_5_2_matmul_opt/`**

- **Purpose:** **matrix multiplication** on MLU using **Bang** sources (`01_scalar.mlu` … `06_vector_sram_unions_pipe5.mlu`, etc.), with `test.sh` / `env.sh` and a **README** that documents expected **baseline timing** behavior across optimization stages (scalar → NRAM → vectorized blocks → pipelining / memory hierarchy tweaks).
- **Outcomes:** staged **performance engineering** of matmul kernels, validated by the provided test script.

---

### Lab 8 (`实验八/`) — three alternative tracks

Official topic files exist under `实验题目/实验八/` for each branch; the answer tree mirrors them.

**1. LLAMA 3.2 (`llama3.2/`)**

- **Goals (from course topic file):** understand **Llama 3.2** (including **multimodal** use), **LoRA** fine-tuning (e.g. via SWIFT), **Triton**-based **Flash Attention**, deployment on **DLP**, inference speed evaluation, and optional **operator replacement** in the model stack.
- **Artifacts:** inference scripts (`infer-3b.py`, `infer-11b.py`, …), fine-tuning shell scripts, `flash_attention_triton_opt.py`, `modeling_mllama.py`, etc.

**2. Stable Diffusion (`StableDiffusion/`)**

- **Goals:** **text-to-image**, **image-to-image**, **inpainting** with **Stable Diffusion** on DLP; **Triton Flash Attention**; **attention** module integration; **DDIM** sampling path.
- **Artifacts:** `txt2img.py`, `img2img.py`, `inpainting.py`, `attention.py`, `ddim.py`, `flash_attention_triton_opt.py`.

**3. Code Llama (`CodeLLAMA/`)**

- **Goals:** **code generation**, **infilling**, **instruction**-style usage; compare **custom** Llama inference stack vs **Hugging Face Transformers**; **Flash Attention** via Triton; end-to-end optimized inference.
- **Artifacts:** `Codellama-infer.py`, `model.py`, `generation.py`, `example_*_mlu.py`, `modeling_llama.py`, `flash_attention_triton_opt.py`.

**Common theme for Lab 8:** large **generative** models on **Cambricon DLP**, with optional **kernel-level** attention optimization and **multi-stage** rubric (inference → multimodal or extra tasks → fine-tuning → Triton op → full model integration).

---

## Development & practice track — `开发与实践/实验答案/`

This is **not** under the top-level `实验答案/` folder but is a second **`实验答案`** directory for the **“开发与实践”** module.

### YOLOv5 (`yolo/`)

**Official objectives (from `开发与实践/实验题目/yolo/实验题目.md`):** **MagicMind** migration for **YOLOv5** object detection; complete **quantization calibration**; add a **MagicMind post-processing** fused op; run end-to-end **inference**.

**Artifacts:** `pytorch_yolov5_inference_fusedOp.py`, `yolov5_mm_utils.py`.

---

### Swin Transformer (`swin/`)

**Official objectives:** implement **Plugin Roll** (CNNL) and **Plugin ReLU** (BANG C) as **custom plugins**, build shared libraries for PyTorch integration, then run **Swin V2** model **export** and **inference** with correct outputs.

**Artifacts:** `plugin_roll.*`, `plugin_relu.*`, `kernel_relu.mlu`, `swin_transformer_v2.py`, `pytorch_swin_transformer_inference.py`.

---

### BERT / SQuAD (`bert/`)

**Official objectives:** BERT **inference** on **SQuAD** using **cnnl-extra** large operators via dynamic libraries; progressive completion of **data loading**, **model/eval**, and full **inference** pipeline.

**Artifacts:** `pytorch_bert_inference.py`, `bert.py`.

---

## How to use this document

- Treat it as a **map of the repository**, not a substitute for the **instructor PDF / 实验指导书** referenced inside many `实验题目.md` files.
- **Hardware/software** assumptions (DLP, MLU, CNNL, CNRT, PyTorch versions) appear in the official topic files where available.
- Folder names like `exp_5_*` under **`实验七`** reflect **internal experiment numbering** in the starter code, not a mistake—always follow your course’s naming.

---

## Summary

| Location | Labs / modules covered in code |
|----------|---------------------------------|
| `实验答案/` | **2–8** (no **1** in tree), including **5 (choose 1)** and **8 (choose 1 of 3)** |
| `开发与实践/实验答案/` | **YOLOv5**, **Swin**, **BERT** practice labs |

Together, the course emphasizes **Cambricon DLP** stack skills: **PyTorch** models, **CNNL/MagicMind**, **custom operators** (Bang / extensions), **kernel optimization**, optional **Verilog PE** simulation, and **large-model** inference/finetuning with **Flash Attention**-style acceleration.
