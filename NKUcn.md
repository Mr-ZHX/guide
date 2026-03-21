# Computer Networking Labs — Overview (English)

This repository (`network`) contains coursework for **computer networking** at **Nankai University (NKU)** — Computer Science College, per root `README.md`.

The tree is organized as **`lab1_socket/`**, **`lab2/`**, and **`lab3/`** (with subfolders **`3-1`**, **`3-2`**, **`3-3`**). This document follows **only those paths**. It states **what each lab is for** and **what the programs are meant to demonstrate**. It does **not** explain socket programming or protocol internals. It does **not** copy student reports (e.g. `* report.md`), personal scores, or informal commentary from the root README.

**Platform:** Implementations target **Windows** (**Winsock**, Visual Studio projects for Lab 1; **VS Code** + Makefile workflows described for Lab 3 in the author notes).

---

## Lab 1 — Stream sockets and a chat protocol (`lab1_socket/`)

**Course intent (from `README.md`):** Design a **two-party chat protocol** over **TCP (stream sockets)** with **timestamps** on messages. Document **message types, syntax, semantics, and ordering**. Provide **module design** (division, responsibilities, flow). Implement in **C/C++** using **raw Winsock-style API** only (no high-level socket wrapper libraries). Command-line UI is acceptable with **usage** instructions. Test and submit code + report.

**What the framework contains:** Visual Studio solution with **`server/`** and **`client/`** — a **multi-client TCP server** that **broadcasts** received lines to connected clients, plus matching client code. A local note file (`socket编程笔记.md`) is supplementary, not a formal spec.

**Outcome:** Working **bidirectional chat** (or multi-user relay) behavior consistent with a written protocol description submitted for grading.

---

## Lab 2 — Web server and Wireshark (`lab2/`)

**Course intent:** Deploy a **web server** (OS of your choice in the handout; this repo includes static assets), author a simple **HTML** page with at least **major, student ID, name**, and a **logo**. Fetch the page in a **browser**, capture traffic with **Wireshark**, and **analyze** the HTTP interaction. Submit capture file and report.

**What the framework contains:** **`index.html`** (sample page) and **`capture_result.pcapng`** (sample capture). **`lab2.md`** may hold local notes; treat as non-authoritative unless your instructor adopts it.

**Outcome:** Demonstrable **HTTP GET** (and related) exchange between browser and server, backed by a **packet capture** and short analysis.

---

## Lab 3 — Reliable transfer over UDP (`lab3/`)

**Course intent:** In **user space**, use **datagram sockets (UDP)** to build **reliable, connection-oriented file transfer** with **error detection**, **ACKs**, **retransmissions**, and stated **flow / congestion** policies. Transmission is **unidirectional**. Each sub-task expects **protocol write-ups**, **timing and throughput** reporting, and (for the full course) **performance experiments** under varying **delay and loss** — see syllabus text in root `README.md`.

### Sub-lab 3-1 (`lab3/3-1/`)

**Goal:** **Stop-and-wait** flow control: connection setup (e.g. SYN/ACK-style flags in the custom header), **checksum** for error detection, **ack + retransmit**, transfer of a **given test file**.

**Framework:** **`UDPsend/send.cpp`** and **`UDPreceive/receive.cpp`**, **Makefile**, sample payload **`helloworld.txt`**.

### Sub-lab 3-2 (`lab3/3-2/`)

**Goal:** Replace stop-and-wait with a **fixed-size sliding window**, **cumulative acknowledgments**, same reliability goals for the test file.

**Framework:** Same layout as 3-1 (sender/receiver, Makefile, test data).

### Sub-lab 3-3 (`lab3/3-3/`)

**Goal:** Extend the sliding-window solution with a **congestion-control policy** (standard algorithm or a justified variant) and still complete the mandated file transfer with metrics.

**Framework:** Same layout as 3-1/3-2.

### Sub-lab 3-4 (described in `README.md` only)

The root README describes **performance comparison experiments** (stop-and-wait vs sliding window; window size sweep; with vs without congestion control) under controlled **delay/loss**. There is **no `lab3/3-4/` directory** in this repository—treat that item as **course specification** you would implement or document separately.

**Outcome (for 3-1 … 3-3 in-repo):** Programs that exchange a file over UDP with the reliability features required for each stage; logs or prints for **transfer time** and **average throughput** as required by the handout.

---

## Summary table (folders present)

| Path | Topic |
|------|--------|
| `lab1_socket/` | TCP chat / relay with Winsock |
| `lab2/` | HTML page + Wireshark capture artifacts |
| `lab3/3-1/` | UDP + reliability + **stop-and-wait** |
| `lab3/3-2/` | UDP + **sliding window** + cumulative ACK |
| `lab3/3-3/` | UDP + sliding window + **congestion control** |

---

*Grading weights and submission rules are defined by NKU course staff; the numeric example table in the repo README is author-specific and is omitted here.*
