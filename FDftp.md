# FDFTP — Reliable File Transfer Protocol (English Lab Manual)

**Fudan University (FuDan Mini Networking) — Reliable Transport & Congestion Control**

This is a complete English lab manual for **FDFTP**. The project implements a reliable file transfer system in **Python 3.6** on top of **UDP**, providing:

- **Reliability** (connection handshake, acknowledgements, timeout retransmission, sequence numbers, and handling duplicate/old packets).
- **Congestion control** based on **TCP Reno**, including **slow start (SS)**, **congestion avoidance (CA)**, and **fast retransmit/fast recovery (FR/FR)**.
- **Dynamic RTT measurement** to adjust **timeout** and **send window (rwnd)** during the transfer.
- **Throughput and packet loss statistics** for upload/download performance.
- **File integrity verification** using **md5sum** (as stated in the project requirements).

---

## 1. Requirements

1. **Python Version**: Python **3.6**
2. Implement:
   - A **server program** that can serve **multiple clients concurrently**.
   - A **client program** that can upload and download **large files** via command-line commands.
3. Handle unreliable networks (packet loss, timeouts, and out-of-order delivery) using an algorithm chosen from course materials.
4. Implement a congestion control algorithm chosen from course materials:
   - **TCP-Reno** style congestion control.
5. Record:
   - Average upload/download speed
   - Packet loss rate
6. Verify file integrity after transfer using **`md5sum`**.

---

## 2. Project Components

Key files:

- `config.py`  
  Configuration constants (server IP/port, MSS/BUFSIZE, default timeouts, initial cwnd, receiver window limits, and debug/performance flags).

- `FDFTPsocket.py`  
  A task helper class used for measuring throughput and score.

- `rdt.py`  
  Core implementation of reliable transport and congestion control.
  - Defines `rdt` base class, used by both client and server.
  - Implements packet creation/parsing, reliable send/receive logic, timeout handling, Reno-style cwnd updates, and dynamic RTT estimation.

- `client.py`  
  Defines `Client` (handshake, command dispatch, file upload/download over `rdt`).

- `server.py`  
  Defines `Server`, `WelcomeServer` (first handshake and port allocation), and `ConnectionServer` (per-client connection handling).

---

## 3. Deployment & Setup

### 3.1 File Layout

The implementation expects a structure that separates client and server files.

#### Deploy both client and server

Assume you have a directory named `code/`:

```plain
code/
  client/
    temp/                # created at deploy time
    client.py
    config.py
    FDFTPsocket.py
    rdt.py
  server/
    temp/                # created at deploy time
    server.py
    config.py
    FDFTPsocket.py
    rdt.py
```

Put upload files in `client/` and download destination files in `server/`.

#### Deploy only client

```plain
code/
  client/
    temp/
    client.py
    config.py
    FDFTPsocket.py
    rdt.py
```

#### Deploy only server

```plain
code/
  server/
    temp/
    config.py
    FDFTPsocket.py
    rdt.py
    server.py
```

### 3.2 Configuration

Edit `config.py`:

**Required**

- `SERVER_IP`  
  IP address of the server (must be reachable from the client).
- `MAX_BANDWIDTH_Mbps`  
  Server maximum bandwidth in Mbps; used by the client to compute a dynamic receive window.

**Optional**

- `SERVER_PORT`  
  Welcome socket port for server; change only if port conflicts.
- `DEBUG`  
  Verbose debug prints.
- `DYNAMIC`  
  Whether to output/enable dynamic RTT measurement logic (default `True` in the provided configuration).
- `PERFORMANCE`  
  Whether to print throughput and loss statistics (default `True`).

**Do not change (safety constraints)**

- `MSS` (max payload size)
- `BUFSIZE` (socket receive buffer size)

---

## 4. Running Instructions

### 4.1 Start the Server

From the `code/` folder:

```bash
python server.py
```

### 4.2 Start the Client(s)

In each client terminal (multiple clients allowed in parallel):

```bash
python client.py
```

After a successful connection handshake, the client prints available commands:

- `upload  : fsnd filename`
- `download: frcv filename`
- `exit    : input nothing:)`

### 4.3 Client Commands

#### Upload (client -> server)

```bash
fsnd filename
```

- `filename` is a relative path under the `client/` directory.
- If the file does not exist, the client prints an error and exits.
- After success, the file will appear under `server/` with the same base name.

#### Download (server -> client)

```bash
frcv filename
```

- `filename` is a relative path under the `server/` directory.
- If the file does not exist, the client prints an error and exits.
- After success, the file will appear under `client/`.

#### Exit Client

- Press `Enter` on an empty line (`line == ''`).
- The client sends a `shutdown` command file to the server.
- Temporary files in `client/temp` and `server/temp` are cleaned by the client logic.

### 4.4 Exception Handling (Deadlock/Handshake Issues)

The project notes that handshake may occasionally fail due to packet loss:

- **Handshake 1**: if the server does not receive `issyn`, the client may block. Terminate that client terminal.
- **Handshake 2**: server responds but client misses the response; terminate that client.
- **Handshake 3**: if the client does not receive an expected response from server, it can still proceed because the client uploads a small file containing measured delay and computed receiver window.

If a client hangs during disconnection:

- The client sets a 1-second socket timeout.
- If repeated transmissions persist, it stops.

In all cases, killing the stuck client should not affect the server.

---

## 5. Transport Protocol Design

This project implements **reliable, ordered file transfer** and **TCP-Reno-like congestion control** over **UDP**.

### 5.1 Packet Format

`rdt.make_pkt` packs a packet using:

- fields: `length`, `seq`, `ack`, `isfin`, `issyn`, `txno`, and `data`

The logical C-like structure (from the project README):

```c
typedef struct {
    int length;   // effective payload length in bytes
    int seq;      // sequence number of current packet
    int ack;      // acknowledgement number
    int isfin;    // end-of-transmission flag (1)
    int issyn;    // handshake flag (1)
    int txno;     // transaction id (incremented per upload/download)
    char[MSS] data; // payload
} packet;
```

### 5.2 Key Semantics

- `seq`: identifies the payload segment position during file transfer.
- `ack`: cumulative acknowledgement (GBN-like / TCP-like behavior).
- `issyn`: used during connection establishment (3-way handshake).
- `isfin`: signals end of file transmission; triggers shutdown/termination logic.
- `txno`: distinguishes different file transfers on the same connection.
  - If receiver sees a mismatched `txno`, it discards and sends a finish ack.

---

## 6. Core Implementation: `rdt.py`

### 6.1 Packet Creation / Parsing

- `make_pkt(...)`: packs a transport packet into bytes.
- `extract(rcvpkt)`: unpacks received bytes back into fields.

### 6.2 Interface between Layers

The reliability layer separates concerns:

- Application -> Transport:
  - `__rdt_send(size)`: reads `size` bytes from file.
- Transport -> Network:
  - `udt_send(sndpkt, addr)`: UDP `sendto`.
- Network -> Transport:
  - `rdt_rcv()`: UDP `recvfrom`.
- Transport -> Application:
  - `__deliver_data()`: writes received delivered bytes into the output file.

### 6.3 Reliable Data Transfer

The send/receive logic is implemented with threads:

- Sender uses:
  - `__send_msg_pkt`: sends packets within the allowed sending window.
  - `__receive_ack_pkt`: receives ACKs and updates cwnd and send-base.
- Receiver uses:
  - `__receive_msg_pkt_and_send_ack_pkt`: receives packets, buffers out-of-order segments, delivers in order, and sends ACKs.

#### 6.3.1 Sender: `__send_msg_pkt`

Main ideas:

- Maintain congestion-controlled send window `cwnd` (TCP Reno behavior).
- Maintain receiver-advertised window `rwnd`.
- Effective window is `min(cwnd, rwnd)`.
- Use **GBN-like behavior** for practical reliability:
  - retransmit based on timeout
  - ACK reflects the “last correctly received in-order segment” (cumulative).

Timeout handling:

- If `time.time() - timer` exceeds `timeout`, treat it as loss:
  - disable RTT measurement for this round
  - reset duplicate ACK counter
  - set `ssthresh = max(cwnd/2, DEFAULT_CWND)`
  - reset `cwnd` to `DEFAULT_CWND`
  - set `send_base` back to current base and retransmit from there
  - double the timeout interval (`timeout *= 2`) for robustness

RTT measurement trigger:

- Every 100 successfully sent packets, the sender sets:
  - `rtt_target_seq = sended`
  - `measure_rtt = True`
  - `rtt_start = time.time()`

Duplicate ACK logic is handled in `__receive_ack_pkt` (see below).

#### 6.3.2 Sender: `__receive_ack_pkt` (TCP Reno)

Key behaviors:

1. **Ignore old transactions**:
   - discard packets if `txno != transaction_no`.
2. **Invalid ACK**:
   - if `ack < send_base - 1`, ignore.
3. **Duplicate ACK**:
   - When `ack == send_base - 1`, increment `dulplicate_ack`.
   - On **3 duplicate ACKs**:
     - set `ssthresh = max(floor(cwnd/2), DEFAULT_CWND)`
     - set `cwnd = ssthresh + 3`
     - move to FR state
     - retransmit `send_base` quickly (fast retransmit).
   - For additional duplicates, increment cwnd slightly (`cwnd += 1`).
4. **Valid ACK**:
   - If RTT measurement is enabled and `ack == rtt_target_seq`, update:
     - `estimate_rtt` using EWMA
     - `dev_rtt` using EWMA of deviation
     - `timeout_interval = estimate_rtt + 4 * dev_rtt`
   - Update receiver window `rwnd` based on measured timeout interval and bandwidth:
     - `rwnd = floor(MAX_BANDWIDTH_Mbps * 1e6 * timeout_interval / (8 * MSS))`
   - Update `send_base` using cumulative ACK:
     - `gap = ack - send_base + 1`
   - Enter/adjust cwnd depending on current state:
     - SS: `cwnd += 1` until exceeding ssthresh, then switch to CA
     - CA: `cwnd += 1/cwnd` per ACK (classic congestion avoidance style)

Termination:

- When `isfin == 1`, sender stops sending and terminates the send thread.

#### 6.3.3 Receiver: `__receive_msg_pkt_and_send_ack_pkt`

Receiver runs in a loop:

- Extract `(length, seq, ack, isfin, issyn, txno, data)`
- Discard old transactions:
  - if `txno` mismatches, send `isfin=1` ack and continue.
- If `isfin == 0`:
  - If `seq` is within receiver window (`seq < expectedseqnum + rwnd`):
    - If `seq < expectedseqnum`: duplicate segment; ACK the last in-order segment.
    - If `seq == expectedseqnum`:
      - buffer the segment
      - deliver all consecutively received segments starting at expectedseqnum
      - write delivered bytes to file (in chunks controlled by `WRITE_MAX`)
      - update `expectedseqnum`
    - Else:
      - buffer out-of-order segments (if not already acked)
    - Send ACK (cumulative): usually `ack = expectedseqnum - 1`.
- If `isfin == 1`:
  - Handle final segment:
    - deliver remaining buffered data if `seq == expectedseqnum`
    - send final ACK and break on finish ack

Window behavior:

- The receiver uses a fixed-size buffering window list:
  - `receive_buffer = [None] * DEFAULT_RWND`
  - `receive_acked = [False] * DEFAULT_RWND`

---

## 7. File Transfer Logic

### 7.1 Upload: `rdt_upload_file(source_path, addr, is_temp_file=False)`

Main steps:

1. Increment `transaction_no`.
2. Determine file size and packet count:
   - `PACKETS_NUM = FILE_SIZE // MSS + 1`
   - `LAST_PACKET_SIZE = FILE_SIZE - (PACKETS_NUM - 1) * MSS`
3. Initialize sender state:
   - `send_base = 1`, `nextseqnum = 1`
   - `cwnd = DEFAULT_CWND`, `ssthresh = rwnd`
   - RTT estimation variables (`estimate_rtt`, `dev_rtt`, etc.)
4. Create threads:
   - one thread for sending data segments (`__send_msg_pkt`)
   - one thread for receiving ACKs (`__receive_ack_pkt`)
5. If it is a “real file” (not a temp command file), enable `Task` measurement:
   - throughput and score printed at the end

### 7.2 Download: `rdt_download_file(dest_path, addr)`

Main steps:

1. Remove previous file if exists.
2. Create an output file in binary mode.
3. Initialize receiver state:
   - `expectedseqnum = 1`
   - receiver buffers and delivered counters
4. Receive segments and send ACKs via receiver thread logic
5. If file ended up empty, remove it.

### 7.3 Shutdown and Cleanup

- The `close()` method closes the UDP socket and removes temp files safely.
- Client sends a special temp file containing `shutdown` so server can stop the connection thread.

---

## 8. Client Logic: `client.py`

### 8.1 Connection Handshake (`connect`)

The client performs a 3-step handshake with:

1. **Handshake 1 (welcome socket)**:
   - send packet with `seq=1`, `issyn=1`
2. **Handshake 2**:
   - receive server reply containing a connection port
3. **Handshake 3 (connection socket)**:
   - send packet with `seq=2`, `issyn=1`
   - measure RTT during the first two steps:
     - compute `timeout = rtt_end - rtt_beg`
     - compute `rwnd` using `MAX_BANDWIDTH_Mbps`, `timeout`, and `MSS`
   - send an upload of a small temp file containing:
     - `timeout rwnd`

After successful handshake, the client prints server address, timeout, and `rwnd`.

### 8.2 Command-based File Transfer (`rdt_transfer`)

When user inputs:

- `fsnd filename`: upload from `client/filename` to `server/...`
- `frcv filename`: download from `server/filename` to `client/...`

Implementation detail:

- For each command, the client performs two file transfers:
  1. Upload a **temporary command file** to server:
     - stored under `client/temp/<client_port>.txt`
     - contains either `fsnd <filename>` or `frcv <filename>`
  2. Transfer the actual file payload using `rdt_upload_file` / `rdt_download_file`.

### 8.3 Shutdown (`shutdown`)

- Write a temp file with content `shutdown`
- Upload it via `rdt_upload_file`
- Client sets socket timeout to 1 second to prevent hanging on repeated FIN transmissions.

---

## 9. Server Logic: `server.py`

### 9.1 Welcome Socket (`WelcomeServer`)

Responsibilities:

- Accept handshake 1 (`issyn=1, seq=1`)
- Reply handshake 2 with a newly allocated `connection_port`
- Spawn a thread for each connection.

It allocates ports using:

- `self.connection_port = int(SERVER_PORT) + 1`
- then increments for each new client.

### 9.2 Per-Client Connection (`ConnectionServer`)

Responsibilities:

- `connect()`:
  - receive client temp file containing `timeout rwnd`
  - parse the values into `self.timeout` and `self.rwnd`
- `rdt_transfer()`:
  - receive temp command file from client (two times: command then actual transfer request)
  - parse command:
    - `fsnd` means client uploads to server
    - `frcv` means client downloads from server
  - call `rdt_upload_file` or `rdt_download_file` accordingly

### 9.3 Parallelism

Server starts:

- one “welcome thread” loop accepting clients
- for each accepted connection, a new thread runs `communicate(connection_port)`
- each connection thread handles file transfers independently.

---

## 10. Performance Measurement & Scoring

### 10.1 Throughput and Loss Statistics

The project uses `Task` to measure:

- `goodput = file_size / (time_consume * 1000)` in Kbps
- `rate = file_size / byte_count`
- printed `score = goodput * rate`

Packet loss rate is approximated:

- `pkt_loss_rate = resend / (resend + sended) * 100%`

### 10.2 Integrity Verification

After transfer completes, verify correctness using:

- `md5sum <file>`

Compare checksums between client and server copies.

---

## 11. Example Real-Environment Test (from project README)

Network setup:

- One Alibaba Cloud server (Hong Kong node), 2 cores / 2G memory, bandwidth 30 Mbps
- Local client machine

Test:

- Upload a ~166MB `cifar-10-python.tar.gz`

Reported results (sample):

- `goodput: 2257.12 Kbps`
- `score: 2205.85`
- `pkt_loss_rate: 0.5958%`
- File verified correct using `md5sum`.

---

## 12. Conclusion

FDFTP demonstrates an end-to-end reliable and congestion-controlled file transfer system using:

- UDP-based reliability mechanisms (sequence/ack, retransmission, RTT-based dynamic timeout)
- TCP-Reno congestion control logic (SS/CA/FR)
- out-of-order buffering for receiver reliability (GBN-like reliability updated toward TCP behavior)
- performance measurement and integrity verification.

---

## Appendix: Quick Command Summary

- Start server:
  - `python server.py`
- Start client:
  - `python client.py`
- Upload:
  - `fsnd filename` (from `client/`)
- Download:
  - `frcv filename` (from `server/`)
- Exit:
  - press `Enter` on empty input

