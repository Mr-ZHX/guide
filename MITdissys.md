# MIT 6.824 Distributed Systems — Programming Labs Overview

This document describes the **programming labs** reflected in this repository ([MIT 6.824](https://pdos.csail.mit.edu/6.824)–style **Go** assignments). It is meant for readers who have **not** taken the course: **what each lab is for** and **what behavior the finished system must provide**. It does **not** explain how to implement the labs and does **not** reproduce student write-ups.

**About this tree:** The repo is a reorganized fork (see root `README.md`) based on **`6.824-golabs-2021`**, with tests and module paths adjusted. **Official policies** (e.g. keeping solutions private) apply; the course staff request that solutions not be published.

**How tests are run (high level):** Most labs use `go test` under `pkg/<lab>/`. MapReduce uses shell scripts under `cmd/mr/scripts/`.

---

## Lab 1 — MapReduce

**Goal:** Build a **MapReduce** system similar in spirit to the original: a **coordinator** schedules **map** and **reduce** tasks over input files; **worker** processes run map and reduce **plugins** (e.g. word count), write intermediate and final outputs, and report back.

**What you implement:** RPC-based cooperation between one coordinator and multiple workers: **task assignment**, **parallelism**, **completion detection**, and **fault tolerance** when workers crash or finish slowly (tasks must eventually complete without manual intervention). The coordinator should expose a way for the driver to know when the **whole job is done**.

**Applications (provided):** Example binaries such as **word count**, **inverted index**, and timing/crash-stress plugins validate scheduling, correctness of outputs, and robustness.

**Success criteria:** The supplied test scripts (`test-mr.sh`, optional `test-mr-many.sh`) pass—covering basic jobs, many files, crashes, parallelism limits, and edge cases defined by the course tests.

---

## Lab 2 — Raft (consensus module)

**Goal:** Implement **Raft** as a **library** used by replicated state machines: a cluster of servers agrees on a **total order of log entries** despite **failures** and **partitioned networks**, electing a **leader** per term and maintaining **safety** (never committing conflicting histories).

The lab is split into three graded parts.

### Lab 2A — Leader election

**What the system must do:** Among a fixed set of peers, servers **elect at most one leader** at a time when enough nodes are connected (**quorum**). If the leader fails or is partitioned away, a **new leader** appears in a new **term** when connectivity allows. With **no quorum**, there should be **no leader** (split-brain avoided for the tested scenarios).

**Tests:** Exercises include initial election, re-election after failures, and repeated elections under churn.

### Lab 2B — Log replication

**What the system must do:** The leader **accepts new log entries** from the test harness (or upper layer), **replicates** them to followers, and **commits** them when safely replicated. Followers **append** entries in order; conflicting tails are resolved per Raft rules. **Agreement:** all servers eventually share the same committed prefix.

**Tests:** Basic append, failure during replication, many concurrent appends, etc.

### Lab 2C — Persistence

**What the system must do:** Servers **restart** from disk: **persisted state** must be enough to preserve Raft safety and avoid “forgetting” committed entries. The implementation must **not** write unbounded full-state snapshots on every small change in a naive way that fails performance tests; the lab introduces **efficient persistence** expectations consistent with the handout.

**Tests:** Restart scenarios, persistence of votes/terms/logs, and performance-related checks.

---

## Lab 3 — Fault-tolerant key/value service (built on Raft)

**Goal:** A **replicated key/value store**: clients issue **Get**, **Put**, and **Append** operations; the service remains **correct** under server crashes and network issues, as long as a majority of replicas in a Raft group stay alive and connected.

**What you implement:** A **KV service layer** on top of your Raft log: operations are **serialized through consensus** so that all replicas apply the **same operations in the same order**. Clients must tolerate **wrong-leader** replies and **retry** until they talk to the current leader. **Duplicate detection** is required so that retried operations do not corrupt state (e.g. **at-most-once** semantics for client commands, typically via client/session identifiers in the command log).

### Lab 3A — Without log compaction

**Focus:** End-to-end correctness and **linearizability** of the key/value history under concurrency and failures (the tests use a **linearizability checker**).

### Lab 3B — With log compaction / snapshots

**Focus:** The Raft layer must support **snapshotting** (or equivalent compaction) so the log does not grow forever; the KV layer cooperates to **trim** replicated state safely while preserving correctness.

**Success criteria:** `kvraft` tests pass for 3A and 3B subsets as defined in `test_test.go`.

---

## Lab 4 — Sharded fault-tolerant key/value service

**Goal:** Scale **horizontally** by **partitioning keys into shards** (fixed number of shards, e.g. **10** in the handout), each served by a **separate Raft replica group**. The system must also **reconfigure**: **add** replica groups, **remove** groups, and **move** individual shards between groups **without losing consistency** or serving stale data across moves.

**Major components (two cooperating subsystems):**

### Shard controller (`shardctrler`)

**What it does:** Maintains a **sequence of numbered configurations**. Each configuration records which **replica group IDs** exist and which group owns each **shard**. Supported operations (conceptually): **Join** (introduce new groups), **Leave** (remove groups), **Move** (transfer one shard’s assignment), and **Query** (fetch a configuration by number, or the **latest**).

**Requirements:** Updates go through **Raft** for fault tolerance; clients must handle **not-leader** redirects. Shards should stay **balanced** across groups (within the tolerance enforced by tests).

### Sharded KV servers (`shardkv`)

**What they do:** Each replica group runs your **Lab 3–style** Raft-backed KV machine **for the shards it owns**. On **configuration changes**, groups must **hand off** shard data so that keys always live on exactly one primary group for the current config, and **duplicates** during migration are handled safely.

**Tests:** Include **static sharding** (no movement), **join/leave** transitions, **shard movement**, concurrent clients, and checks that **linearizability** holds where required.

---

## Optional / legacy command trees

The repository layout may still contain binaries under `cmd/` (e.g. older coursework names like **view service** / **primary-backup** demos). The **official checklist** in this repo’s `README.md` centers on **Labs 1–4** above; treat other `cmd/` targets as **out-of-band** unless your instructor assigns them.

---

## Summary table

| Lab | Topic | Deliverable (behavioral) |
|-----|--------|---------------------------|
| **1** | MapReduce | Coordinator + workers; crash-tolerant batch processing |
| **2A** | Raft election | Stable leadership & terms under failures |
| **2B** | Raft log | Replicated, committed log; agreement |
| **2C** | Raft persistence | Correct recovery after restart |
| **3A** | KV + Raft | Linearizable Get/Put/Append |
| **3B** | KV + compaction | Same as 3A with bounded log growth |
| **4** | Sharding | Shardctrler + shardkv; safe reconfiguration & migration |

---

*For exact deadlines, collaboration rules, and hidden tests, refer to the current term’s [6.824 schedule](https://pdos.csail.mit.edu/6.824) and handouts. This overview is aligned with the lab names and packages present in this repository.*
