# HIT 2024 Fall Computer Network — English Lab Manual

This document summarizes the 5 labs contained in `HIT-2024fa-computer-network/` based on the repository README and the provided source code. It focuses on objectives, design/implementation notes, build/run steps, and verification ideas.

## Project overview

The repository contains the following labs:

1. `lab1`: Design and implement an HTTP proxy server
2. `lab2`: Design and implement a reliable data transfer protocol (GBN)
3. `lab3`: Use Wireshark for protocol analysis
4. `lab4`: Forward and receive IP datagrams
5. `lab5`: Build and configure a simple network

Lab 1 and Lab 2 are implemented in Java (Windows 11 + Java 21 + Maven). Lab 4 is implemented in C (Debian VM + gcc + gdb, with raw sockets). Lab 3 and Lab 5 have no code in `src/` (they rely on protocol analysis and system/network configuration, respectively).

## Recommended environment (from `README.md`)

Lab 1/2:
- OS: Windows 11 23H2
- Language: Java 21
- IDE: IntelliJ IDEA 2024.2.4
- Build tool: Apache Maven 3.9.6

Lab 4:
- Virtual machine: VMware Workstation Pro 17.5.0
- VM OS: Debian live 12.7.0 amd64 standard, Linux kernel 6.1.x
- Host OS: Windows 11 23H2
- Language: C 23
- Build/debug: gcc 12.x, GNU gdb
- Network tools: netplan.io, TShark/Wireshark, ufw

## Lab 1 — HTTP Proxy Server

### Goal

Implement an HTTP proxy server that:

- Accepts TCP client connections
- Parses HTTP requests/responses
- Forwards requests to the target server and returns responses to the client
- Applies access control (blocking) and site redirection
- Implements a simple HTTP caching mechanism using `Last-Modified` and `If-Modified-Since`

### Implementation summary (code structure)

- Entry point: `src/lab1/src/main/java/cn/edu/hit/server/HttpProxyServer.java`
- Per-connection handler: `ProxyHandler.java`
- HTTP parsing and forwarding: `utils/HttpUtils.java`
- Blocking/redirection rules: `filter/FilterManager.java`
- Cache: `cache/CacheManager.java`, `cache/CacheEntry.java`
- HTTP data model:
  - `core/HttpRequest.java`
  - `core/HttpResponse.java`
  - `core/HttpConstant.java`, `core/HttpStatus.java` (status codes)

### Key behaviors from the code

1. Concurrency
- `HttpProxyServer` uses a fixed thread pool (default size `20`) and listens on port `20000` by default.
- Each accepted connection is handled by `new ProxyHandler(...)` submitted to the thread pool.

2. Supported protocol (HTTP only)
- `ProxyHandler` rejects non-HTTP traffic by checking whether the request port matches `HttpConstant.HTTP_DEFAULT_PORT` (`80`).

3. Filtering and redirection
- The default `main` hardcodes:
  - blocked site: `www.hit.edu.cn`
  - blocked user (client IP): `127.0.0.2`
  - redirection: `jwes.hit.edu.cn` -> `jwts.hit.edu.cn`
- `ProxyHandler` checks:
  - if the client IP is blocked, it stops immediately
  - if the `Host` value is blocked, it stops immediately
  - if a redirect mapping exists, it updates the request `Host` header before forwarding

4. Request/response parsing and forwarding
- `HttpUtils.parseHttpHeader(...)` reads bytes until it finds `\r\n\r\n`.
- Request parsing:
  - Parses request line: `METHOD URI VERSION`
  - Parses headers as `key: value`
  - Reads body only when `Content-Length` is present
- Response parsing:
  - Parses status line: `HTTP_VERSION STATUS_CODE STATUS_TEXT`
  - Parses headers
  - Reads body using either:
    - `Content-Length` (fixed length)
    - `Transfer-Encoding: chunked` (chunked decoding implemented)

- Forwarding:
  - For the request, it forwards `request.toString()` (headers + blank line) and then forwards the body when `Content-Length` exists.
  - For the response, it forwards `response.toString()` and then forwards the body using:
    - chunked transfer encoding when `Transfer-Encoding: chunked` exists, otherwise
    - fixed-length body when `Content-Length` exists

5. Caching logic (conditional GET style)
- `CacheManager` stores serialized `HttpResponse` objects on disk under a local `cache/` directory in `user.dir`.
- `ProxyHandler` uses the request URI string as the cache key:
  - If cache hit:
    - it adds `If-Modified-Since` using the cached response’s `Last-Modified`
  - After forwarding:
    - if the server returns `304 Not Modified` and cache hit exists, it returns the cached response directly
    - if `Last-Modified` exists in the new response, it updates the cache

### Build and run

Build:
- Open `src/lab1/` in IntelliJ.
- Ensure Maven downloads dependencies (only `commons-io` is declared).
- Build/run `HttpProxyServer`.

Run:
- Start `HttpProxyServer` via its `main`.
- Default listening endpoint: `127.0.0.1:20000`.

Test (typical workflow):
- Configure your browser or HTTP client to use `HTTP Proxy: 127.0.0.1:20000`.
- Send HTTP requests to:
  - a normal site (should be forwarded)
  - `www.hit.edu.cn` (should be blocked)
  - `jwes.hit.edu.cn` (should be redirected by changing `Host`)

Caching verification idea:
- Visit the same URI twice.
- On the second visit, you should observe that the proxy adds `If-Modified-Since` and handles `304 Not Modified` by serving cached content.

### Submission/verification checklist (suggested)

1. Proxy starts correctly and listens on the expected port.
2. Forwards HTTP requests to the correct `Host` and port.
3. Blocking/redirect logic triggers as expected.
4. Cache behavior:
   - first request populates cache if `Last-Modified` exists
   - subsequent request uses `If-Modified-Since`
   - `304` results in cached response being returned

## Lab 2 — Reliable Data Transfer (GBN)

### Goal

Implement a reliable file/data transfer protocol on top of UDP that uses the Go-Back-N (GBN) sliding window concept and supports:

- Packetization with a fixed payload size
- Sequence numbers with wrap-around (modulo)
- Cumulative ACK behavior (receiver ACK indicates the next expected behavior in the sender logic)
- Timeout-based retransmission of the window

### Packet and ACK format (from code)

Core constants are in `src/lab2/src/main/java/cn/edu/hit/config/CommonConfig.java`:
- `DATA_SIZE = 1024`
- `ACK_SIZE = 1`
- `TIMEOUT = 1000` (ms)

`Packet` (`core/Packet.java`):
- `seqNum`: 1 unsigned byte stored as int (`0..255`)
- `data`: fixed-size payload of `1024` bytes in the serialized form
- `dataLength`: 2 bytes (unsigned short stored via `putShort`)
- `eof`: 1 byte flag (`1` means EOF packet)

Serialized packet size:
- `CommonConfig.BUFFER_SIZE = 1028`, which matches: `1 + 1024 + 2 + 1 = 1028`.

`ACK` (`core/ACK.java`):
- `seqNum`: 1 byte ACK (also limited to `0..255`)

GBN parameters are in `GBNConfig.java`:
- `SERVER_PORT = 12340`
- `WINDOW_SIZE = 5`
- `SEQ_BITS = 3` so `seqSize = 2^3 = 8`

### Implementation summary (GBN sender/receiver)

GBN Sender: `src/lab2/.../core/GBNSender.java`
- Uses UDP `DatagramSocket` to send `Packet` bytes to the client address/port.
- Maintains:
  - `base`: the sliding window base (packet index, not only modulo seq)
  - `nextSeqNum`: next packet index to send
  - `sentPackets`: map from `seqNum % seqSize` to the last sent `Packet` for that sequence number
  - `Timer`: schedules timeout retransmission

Send loop (`sendData`):
- Compute totalPackets as `ceil(data.length / DATA_SIZE)`.
- While `base < totalPackets`:
  - Send packets while:
    - `nextSeqNum < base + windowSize` and
    - `nextSeqNum < totalPackets`
  - Mark last packet with `eof = true` when `nextSeqNum == totalPackets - 1`.
  - Simulate packet loss on sending with `packetLossRate` (from `CommonConfig.SENDER_PACKET_LOSS_RATE = 0.1`).
  - Start timer when the first unACKed packet is sent (`base == nextSeqNum`).
  - Wait for ACKs using `receiveAck()`.

ACK handling:
- It receives ACKs until `windowSlide(ackNum) > 0`.
- It advances `base` by `slideSize`.
- Stops the timer if the window is fully acknowledged (`base == nextSeqNum`), otherwise restarts it.

Timeout retransmission:
- On timeout, retransmit all packets in `[base, nextSeqNum)` by iterating indexes and using `sentPackets.get(i % seqSize)`.

GBN Receiver: `src/lab2/.../core/GBNReceiver.java`
- Maintains `expectedSeqNum` and `eof`.
- For each received UDP packet:
  - If `packet.seqNum == expectedSeqNum`:
    - append payload (`packet.getData()`) to output file
    - send cumulative ACK `expectedSeqNum` (optionally dropped to simulate ACK loss)
    - increment `expectedSeqNum` modulo `seqSize`
    - if `packet.eof` then stop
  - Else (out-of-order):
    - send ACK for the previous expected value: `(expectedSeqNum - 1 + seqSize) % seqSize` (optionally dropped)

### Run instructions (interactive App)

The interactive application is `src/lab2/src/main/java/cn/edu/hit/app/App.java`.

There are two provided entry points for running the two communicating nodes:
- `src/lab2/src/test/java/TestNodeOne.java`
  - uses `srcPort = 20001`, `dstPort = 20002`
- `src/lab2/src/test/java/TestNodeTwo.java`
  - uses `srcPort = 20002`, `dstPort = 20001`

Recommended workflow:
1. Start `TestNodeOne` in one console/IDE run configuration.
2. Start `TestNodeTwo` in another console/IDE run configuration.
3. In `TestNodeTwo` (client download mode), type:
   - `-testgbn prince.txt` to test GBN
   - or `-testsr prince.txt` to test SR (Selective Repeat, implemented as well)

Where files are:
- Upload (server): `assets/upload/<fileName>`
- Download (client output): `assets/download/<fileName>`

Important note about file output:
- `IOUtils.appendBytesToFile(...)` opens the output file in append mode.
- Before each new test, delete/clear `assets/download/<fileName>` to avoid concatenating multiple runs.

### Verification ideas (what to observe)

1. Correctness:
- After transfer finishes (EOF), the received file should match the sent file contents.

2. Sliding window and retransmission:
- Under simulated loss (code uses loss rate 0.1), you should see repeated transmissions due to timeouts.

3. ACK behavior:
- Receiver ACKs should progress when expected packets arrive.
- Out-of-order packets should trigger “duplicate/previous ACK” responses.

4. Wireshark correlation (if used in lab3):
- Data packets should have size 1028 bytes (serialized packet).
- ACK packets should have size 1 byte.
- Timeout-based retransmissions should roughly align with `TIMEOUT = 1000ms`.

## Lab 3 — Wireshark Protocol Analysis

### Goal

Use Wireshark to capture and analyze protocol behavior for:
- Lab 1: HTTP proxy forwarding and caching validation behavior
- Lab 2: UDP reliable transfer packet/ACK behavior (GBN)

### Analysis guidance (suggested)

HTTP proxy analysis (Lab 1):
- Capture traffic while issuing browser requests through the proxy.
- Verify that forwarded requests preserve/override headers correctly:
  - `Host` changes for redirect rules
  - blocked sites generate no outgoing request (proxy terminates early)
- For caching:
  - On the second request for the same URI, you should observe `If-Modified-Since`.
  - If the origin server responds with `304 Not Modified`, the proxy returns cached content without requiring a full new response.

GBN analysis (Lab 2):
- Capture UDP packets between the two nodes running `TestNodeOne`/`TestNodeTwo`.
- Observe:
  - the fixed serialized packet size (1028 bytes)
  - the sequence number changes modulo `2^SEQ_BITS`
  - ACK packets size 1 byte and repeated ACK patterns on loss/out-of-order
  - retransmissions around the configured timeout (1000 ms)

Wireshark filters (examples):
- UDP traffic filter: `udp`
- Filter by ports (replace with your chosen ports):
  - `udp.port == 20001 or udp.port == 20002`

### Submission checklist (suggested)

- At least one capture for each lab (HTTP and GBN).
- Screenshots showing key header fields:
  - HTTP status code transitions (`200` -> `304`)
  - GBN seq/ack evolution and at least one retransmission event

## Lab 4 — IP Datagram Forwarding and Sending/Receiving

### Goal

Implement user-space sending/receiving and forwarding of IP datagrams by:

1. Sending and receiving UDP messages over IP
2. Forwarding packets at the application layer (basic relay)
3. Implementing forwarding at the raw Ethernet/IP layer:
   - parse Ethernet + IP header
   - decrement IP TTL
   - recompute IP header checksum
   - select next-hop MAC address using a routing table
   - rebuild Ethernet header and transmit using `AF_PACKET` raw sockets

### Provided implementations and corresponding sections

The folder names show progressive complexity:
- `sec-4-1-2`: basic send/receive/forward using UDP sockets
- `sec-4-2-2`: forwarding with a static routing table and raw sockets (more complete)
- `sec-4-3-3`: full custom Ethernet+IP packet sending and a next-hop router that updates TTL/checksum

#### `sec-4-1-2` (basic UDP send/receive/forward)

- `send_ip.c`
  - Creates a UDP socket and repeatedly sends user-input messages.
  - Sends to a fixed destination IP and port.
- `recv_ip.c`
  - Binds UDP port and prints received messages with timestamp and source IP/port.
- `forward_ip.c`
  - Receives UDP datagrams on a local port and re-sends them to another fixed destination IP/port.

#### `sec-4-2-2` (raw forwarding with static routing table)

- `myroute.c`
  - Uses `socket(AF_PACKET, SOCK_RAW, htons(ETH_P_IP))` to capture raw IP frames.
  - Defines a `routing_table` with two entries mapping destination networks to:
    - destination network (dest + netmask)
    - outgoing interface (`ens36` or `ens37`)
    - next-hop MAC addresses
  - For each captured frame:
    - parses Ethernet header and IP header
    - looks up matching route by `(dest_ip & netmask) == (route.dest & route.netmask)`
    - decrements `ip_header->ttl` by 1
    - recomputes `ip_header->check` using a standard Internet checksum function
    - updates Ethernet source/destination MAC addresses
    - transmits the modified frame via `sendto()` with `sockaddr_ll` configured from the selected interface

- `mysend.c`, `myrec.c`
  - Provide UDP-based send/receive test endpoints (not the raw router).

#### `sec-4-3-3` (Ethernet+IP packet building and TTL/checksum router)

- `send.c`
  - Builds an Ethernet frame:
    - source MAC from the local interface (`ens33`)
    - destination MAC hardcoded to a next-hop MAC
    - Ethernet type set to IPv4
  - Builds an IPv4 header:
    - sets `ttl`, `protocol = UDP`
    - computes IP header checksum
  - Builds a UDP header and payload
  - Sends the raw frame with `sendto()` on an `AF_PACKET` raw socket.
  - Asks the user for destination IP and message content.
- `recv.c`
  - A normal UDP receiver for port `12345`.
- `route.c`
  - Captures raw IP frames via `AF_PACKET` raw socket.
  - Maintains an exact-match routing table on destination IP strings.
  - For frames:
    - it discards packets whose source IP is not the configured `SRC_IP`
    - decrements TTL and recomputes checksum
    - selects next-hop MAC from the routing table
    - updates Ethernet destination/source MAC and forwards the frame

### Build and run (Debian VM)

Because raw sockets require elevated privileges, run forwarding/sending programs with `sudo`:
- `sudo ./route`
- `sudo ./send`

Compile examples (adjust file names accordingly):
- `gcc -O2 -std=c23 send_ip.c -o send_ip`
- `gcc -O2 -std=c23 recv_ip.c -o recv_ip`
- `gcc -O2 -std=c23 forward_ip.c -o forward_ip`
- `gcc -O2 -std=c23 myroute.c -o myroute`
- `gcc -O2 -std=c23 route.c -o route`
- `gcc -O2 -std=c23 send.c -o send`
- `gcc -O2 -std=c23 recv.c -o recv`

Run order idea (for the most complex section `sec-4-3-3`):
1. Start `recv.c` on the destination host (UDP port `12345`).
2. Start `route.c` on the router host (raw forwarder).
3. Start `send.c` on the sender host (raw packet builder).
4. Verify:
   - forwarded packets arrive at `recv.c`
   - router logs show TTL and forwarding decisions

Important parameters:
- Many IP addresses and MAC addresses are hardcoded in the C code.
- You must align the VM network configuration (interfaces and addresses) to match the constants in the code.

### Verification checklist (suggested)

1. TTL behavior:
- observe TTL decrement by 1 at each forwarding hop (where implemented)
2. Checksum behavior:
- confirm checksum is recomputed after TTL modification (can be validated via Wireshark / packet inspection)
3. Routing decisions:
- confirm routing table lookup selects the expected next hop
4. Discard conditions:
- ensure packets are dropped when TTL expires or when no route entry is found

## Lab 5 — Simple Network Building and Configuration

### Goal

Build a small network topology in a virtual environment and configure:
- IP addressing and subnet masks
- routing (static routes / default routes)
- connectivity verification (ping, UDP tests, etc.)
- packet observation (optional Wireshark/TShark captures)

### Suggested tasks (because Lab 5 has no code in `src/`)

Since the repository code focuses on lab4 behavior, Lab 5 is typically the system setup required to make lab4 and lab2 experiments work in the chosen topology.

Suggested workflow:
1. Create VM network topology (hosts + router) according to the lab guide.
2. Configure VM interface IPs to match the constants used in lab4 code.
3. Configure routing so that:
   - sender can reach receiver
   - router can forward according to its static routing table (for the raw forwarder code)
4. Validate:
   - `ip addr` (interface addresses)
   - `ip route` (routing table)
   - ping from sender to router/receiver
5. Run lab4 programs and take screenshots/logs for acceptance.

## Deliverables (suggested)

Prepare an English submission that includes:
- For Lab 1: proxy design description + screenshots showing forwarding/blocking/redirect/caching
- For Lab 2: explanation of GBN sliding window + sequence/ACK analysis + file transfer correctness evidence
- For Lab 3: Wireshark capture screenshots and field-level explanation
- For Lab 4: forwarding design explanation (TTL/checksum/routing table) + experiment logs/screenshots
- For Lab 5: network topology + IP/routing configuration screenshots + connectivity verification

## End of document

