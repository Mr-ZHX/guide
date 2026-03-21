# Stanford CS144 (Minnow) — Lab Overview

This document summarizes **what each checkpoint expects** in this repository: goals, required behavior at a **black-box** level, and how success is typically checked. It is **not** an implementation guide and does **not** reproduce student writeup templates from `writeups/`.

**Project context:** You build pieces of a user-space **TCP/IP stack** (reliable byte stream, TCP sender/receiver logic, Ethernet/IPv4/ARP glue, routing) and run it over a **TUN** virtual network device. The official course site is [cs144.stanford.edu](https://cs144.stanford.edu).

**Build & test (this tree):**

- Configure: `cmake -S . -B build`
- Build: `cmake --build build`
- Tests: `cmake --build build --target test`
- Named aggregates (from `etc/tests.cmake`): `check0`, `check1`, `check2`, `check3`, `check5`, `check6`, `check_webget`, optional `speed` targets.

---

## Checkpoint 0 — `ByteStream` and HTTP `webget`

### What you do

1. **Environment:** Set up the build system and dependencies so the project compiles and tests run.
2. **`ByteStream`:** Implement an in-memory **FIFO byte stream** with a fixed **capacity**, exposed as separate **Writer** and **Reader** sides.
3. **`webget`:** Complete the small program that performs an **HTTP/1.1 GET** to a given host and path and prints the response body (and headers as received) to standard output.

### Functionality expected (`ByteStream`)

- **Writer:** `push` data up to remaining capacity; track total bytes pushed; `close` to signal end-of-write; report available capacity; reflect an **error** state when set.
- **Reader:** `peek` at buffered data, `pop` to consume bytes; know when the stream is **finished** (closed and fully read); track bytes buffered and total popped.
- Semantics must match the unit tests: basic I/O, capacity limits, many/split writes, stress cases.

### Functionality expected (`webget`)

- Using the course’s **`CS144TCPSocket`** (TCP over IPv4 over TUN), **connect** to the server, send a minimal valid **GET** request with **Host** and **Connection: close**, then **read until end-of-file** and print everything received.

### How it is checked

- CTest targets matching `^byte_stream_` (sanitized and unsanitized binaries).
- Script **`tests/webget_t.sh`:** runs `webget` against a known host/path and compares a **hash** of the last line of output to an expected value.
- CMake target **`check0`** runs `webget` and all `byte_stream_*` tests together.

---

## Checkpoint 1 — `Reassembler`

### What you do

Implement a **Reassembler** that accepts **indexed substrings** of a single logical byte stream, which may arrive **out of order**, **duplicate**, or **overlapping**, and writes the reconstructed stream in order into an underlying **`ByteStream`**.

### Functionality expected

- **`insert(first_index, data, is_last_substring)`:** merge incoming pieces with what is already known; as soon as the next expected byte is known, **write contiguously** to the output stream.
- **Buffering:** Bytes that fit within the stream’s **remaining capacity** but cannot yet be written (gap before them) must be **stored** until the gap fills.
- **Eviction:** Bytes that **cannot** ever be delivered within capacity (beyond the window implied by capacity and already-written prefix) must be **discarded**.
- **End of stream:** After the **last** byte has been written, the output stream should be **closed** appropriately when the reassembly is complete.
- **`bytes_pending`:** Report how much data is held inside the reassembler (not yet written to the output).

### How it is checked

- Tests: `reassembler_single`, `reassembler_cap`, `reassembler_seq`, `reassembler_dup`, `reassembler_holes`, `reassembler_overlapping`, `reassembler_win`.
- **`check1`** = all `byte_stream_*` and `reassembler_*` tests.

---

## Checkpoint 2 — `Wrap32` and `TCPReceiver`

### What you do

1. **`Wrap32`:** Implement **32-bit sequence numbers** that wrap at 2³², including conversion to/from **absolute** sequence numbers and comparison consistent with TCP’s finite window.
2. **`TCPReceiver`:** Consume **incoming TCP segments** (as abstract messages), drive the **`Reassembler`**, and produce **acknowledgements and window information** for the peer’s sender.

### Functionality expected (`Wrap32`)

- **`wrap`:** Map an absolute sequence number and a **zero point** (ISN) to the on-wire 32-bit value.
- **`unwrap`:** Map a 32-bit value back to an **absolute** sequence number **closest** to a given **checkpoint** (resolving ambiguity from wrapping).

### Functionality expected (`TCPReceiver`)

- **`receive`:** Interpret segment messages (SYN, payload with sequence indexing, FIN), update state, and insert payload at the correct indices in the reassembler.
- **`send`:** Return the appropriate **ACK/window** summary for the peer (as structured in `TCPReceiverMessage`), consistent with TCP rules assumed by the tests.

### How it is checked

- Tests: `wrapping_integers_*`, `recv_*` (connect, transmit, window, reordering, close, edge cases).
- **`check2`** = `byte_stream_*`, `reassembler_*`, `wrapping_*`, `recv_*`.

---

## Checkpoint 3 — `TCPSender`

### What you do

Implement the **`TCPSender`**: given a **byte stream of outbound data**, emit **TCP segments** (as `TCPSenderMessage` values) that respect **receive window**, handle **acknowledgements**, perform **retransmissions** with a **retransmission timer** and **exponential backoff**, and correctly open/close the connection (SYN/FIN handling as required by the test suite).

### Functionality expected (black box)

- **`push` / `tick`:** When allowed by window and state, **segmentize** data from the writer; **transmit** via a callback; on timeout, **retransmit** appropriately.
- **`receive`:** Process incoming ACKs; advance **first unacknowledged** sequence; update what may be sent; interact with the timer per the spec implied by tests.
- Track **bytes in flight** and **consecutive retransmissions** as exposed by the API.

### How it is checked

- Tests: `send_*` (connect, transmit, retransmission, window, ack processing, close, extra cases).
- **`check3`** = all tests through `byte_stream_*`, `reassembler_*`, `wrapping_*`, `recv_*`, and `send_*`.

---

## Running TCP over IP — `tcp_ipv4` and `tcp_native` (integration milestone)

This repository does not define a CMake target named `check4`, but it ships **applications** that exercise the **full TCP implementation** end-to-end.

### What you do

With **`TCPSender`**, **`TCPReceiver`**, and the surrounding adapters complete, you use:

- **`tcp_ipv4`:** TCP over **IPv4** through the **TUN** device (client/server modes, optional loss simulation, configurable timeouts and windows).
- **`tcp_native`:** Compare or debug against the **host OS TCP** socket (`TCPSocket`) in a similar client/server pattern.

### Functionality expected

- These programs **bidirectionally copy** streams between TCP and stdin/stdout (or peer), validating that your stack interoperates in realistic settings (including with the OS stack for `tcp_native`).

---

## Checkpoint 5 — `NetworkInterface`

### What you do

Implement the **link-layer / internet-layer boundary**: a **`NetworkInterface`** that sends and receives **Ethernet frames**, encapsulates **IPv4 datagrams**, and implements **ARP** (learning mappings, replying to requests, **queueing** datagrams until an Ethernet address is known).

### Functionality expected

- **`send_datagram`:** Encapsulate an IPv4 datagram in Ethernet; if the **next hop’s Ethernet address** is unknown, **broadcast ARP** and **buffer** datagrams until resolution (subject to timeouts/behavior enforced by tests).
- **`recv_frame`:** Accept frames addressed to this interface; pass **IPv4** payloads up; handle **ARP request** (learn + reply) and **ARP reply** (learn).
- **`tick`:** Model **time**: expire **ARP cache** entries and **pending** ARP / queued-datagram behavior as required.

### How it is checked

- Test: `net_interface`.
- **`check5`** runs `net_interface` only.

---

## Checkpoint 6 — `Router`

### What you do

Implement a **`Router`** with multiple **`NetworkInterface`** objects, a **routing table**, and **`route()`** to forward arriving datagrams to the correct outgoing interface using **longest-prefix match**.

### Functionality expected

- **`add_route`:** Install routes (prefix/length, next-hop optional for directly connected networks, egress interface index).
- **`route`:** For each interface’s received datagrams, **decrement TTL**, validate, select **best route**, and **emit** via the chosen interface (using the **next hop** for link-layer addressing as appropriate).

### How it is checked

- Test: `router` (often in conjunction with stacked interfaces).
- **`check6`** runs `net_interface` and `router`.

---

## Checkpoint 7 — End-to-end peering (`endtoend` and writeup expectations)

### What you do

The **`endtoend`** application (see `apps/endtoend.cc`) builds a **multi-hop Ethernet scenario** with hosts and a router, runs your **TCP over IP** stack over it, and is used for **final integration**: two stacks talking through the **router**, validating **real** forwarding + ARP + TCP together.

The course **writeup** for checkpoint 7 (template in `writeups/check7.md`) asks you to report whether your implementation can **open and close** a conversation with **another student implementation** (or a second instance of your own) and **transfer a large file** (e.g. one megabyte) with **bit-identical** results—without requiring you to paste that questionnaire here.

### Functionality expected

- **Stable TCP** through **multiple Ethernet segments** and **routing**.
- **Interoperability** with another correct implementation (roles: client/server).

### How it is checked

- Primarily **manual / partner** testing plus course grading procedures; the codebase provides **`endtoend`** as the driver scenario.

---

## Optional: speed benchmarks

CMake target **`speed`** runs benchmarks such as **`byte_stream_speed_test`** and **`reassembler_speed_test`** (separate from the default `test` filter that excludes speed tests). These check **asymptotic / throughput** behavior of your stream and reassembler, not just correctness.

---

## Module-to-file map (student implementation files under `src/`)

| Area              | Typical files        |
|-------------------|----------------------|
| Byte stream       | `byte_stream.cc`     |
| Reassembly        | `reassembler.cc`     |
| TCP seq. numbers  | `wrapping_integers.cc` |
| TCP receive path  | `tcp_receiver.cc`    |
| TCP send path     | `tcp_sender.cc`      |
| Link/IP (ARP)     | `network_interface.cc` |
| Routing           | `router.cc`          |
| Socket template   | `tcp_minnow_socket.cc` (explicit instantiations) |

Other directories (`util/`, `tests/`, `apps/`) provide infrastructure, adapters, tests, and demo programs.

---

*This overview is derived from public headers, `etc/tests.cmake`, and application entry points in this tree. Course staff may publish additional policies or hidden tests not reflected here.*
