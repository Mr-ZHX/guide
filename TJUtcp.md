# Experiment Description (English): TCP-like Reliable Transport (TJU TCP)

## 1. Project Overview
This project implements a TCP-like reliable transport protocol over a simulated network. A lightweight "kernel" layer receives UDP datagrams, reconstructs the TCP-like packet, and dispatches it to your TCP stack based on a socket lookup (4-tuple).

Your task is to implement the TCP-like socket behavior and protocol mechanisms so that the provided client/server/test programs can establish connections, reliably transfer data, and close connections.

Key interfaces are declared in `computernetwork_tju_tcp/inc/tju_tcp.h`, and packet formats are defined in `computernetwork_tju_tcp/inc/tju_packet.h`.

## 2. Experimental Requirements
The experiment requirements can be summarized as the following deliverables:

1. Provide a TCP socket abstraction that supports the full life cycle:
   - create a socket
   - bind/listen/accept for the server side
   - connect for the client side
   - send/recv for data transfer
   - close for connection teardown

2. Implement connection establishment and teardown using a TCP-like state machine:
   - three-way handshake for connection establishment
   - four-way teardown (FIN-based) for connection closure, including handling of states such as `TIME_WAIT`

3. Implement reliable, ordered delivery of data:
   - use sequence numbers (`seq_num`) for sent segments
   - use acknowledgements (`ack_num`) to confirm received segments
   - recover from loss using retransmission (timeout-driven and, optionally, duplicate ACK-driven fast retransmit)

4. Implement flow control and congestion control:
   - flow control via an advertised receive window (receiver window and `advertised_window` field)
   - congestion control via a congestion window (`cwnd`) and slow-start/avoidance/recovery behavior (enabled in this project via compile-time macros)

5. Integrate packet processing with the kernel:
   - implement `tju_handle_packet()` to process incoming packets and update socket state, window variables, timers, and retransmission behavior

6. Use the fixed kernel sending interface:
   - the kernel provides `sendToLayer3()` to transmit your TCP-like packets; this function is marked as not modifiable in `src/kernel.c`

## 3. Functional Requirements (What You Need to Implement)
This section lists the functional modules you should implement to satisfy the requirements.

### 3.1 TCP Socket API (required behavior)
Implement the following functions with the semantics described in `inc/tju_tcp.h`:

1. `tju_socket()`
   - Allocate and initialize a TCP control block (TCB)
   - Initialize the socket state to `CLOSED`
   - Prepare internal buffers and synchronization primitives for send/recv

2. `tju_bind(tju_tcp_t* sock, tju_sock_addr bind_addr)`
   - Store the local IP and port used for listening

3. `tju_listen(tju_tcp_t* sock)`
   - Set the socket state to `LISTEN`
   - Register the listening socket into the kernel's listen table (so handshake packets can be dispatched)
   - Initialize server-side queues for partially and fully established connections (as needed by the design)

4. `tju_accept(tju_tcp_t* sock)`
   - Block until the three-way handshake completes
   - Return a connected socket that is ready for `tju_send()` and `tju_recv()`

5. `tju_connect(tju_tcp_t* sock, tju_sock_addr target_addr)`
   - Client-side three-way handshake (send `SYN`, receive `ACK|SYN`, send final `ACK`)
   - Update socket state to `ESTABLISHED` when the handshake succeeds
   - Ensure user code can call `tju_send()`/`tju_recv()` after `tju_connect()` returns

6. `tju_send(tju_tcp_t* sock, const void* buffer, int len)`
   - Provide buffered data for the sender to transmit
   - Cooperate with the sending logic that uses sequence numbers, sliding window bounds, and timers

7. `tju_recv(tju_tcp_t* sock, void* buffer, int len)`
   - Block until data is available in the receiver buffer
   - Read data up to `len` and return the number of bytes copied

8. `tju_close(tju_tcp_t* sock)`
   - Execute FIN-based teardown
   - Trigger appropriate state transitions and retransmission where needed to handle packet loss

### 3.2 Packet Format Usage and Parsing
Use the TCP-like packet header defined in `inc/tju_packet.h`:

1. Header fields
   - `source_port`, `destination_port`
   - `seq_num`, `ack_num`
   - `flags` (e.g., `SYN`, `ACK`, `FIN`)
   - `advertised_window` (receiver-advertised flow control window)
   - `hlen` and `plen` (header length and total packet length)

2. Packet helper functions
   - Use `create_packet_buf()` to build packets
   - Use `get_seq()`, `get_ack()`, `get_flags()`, `get_plen()`, `get_advertised_window()` to parse incoming packets

### 3.3 Connection Establishment (Three-Way Handshake)
Implement handling for the handshake packets in `tju_handle_packet()` and ensure state transitions:

1. Server-side:
   - When in `LISTEN`, receive `SYN` and respond with `ACK|SYN`
   - When in `SYN_RECV`, receive the final `ACK` and move to `ESTABLISHED`

2. Client-side:
   - When in `SYN_SENT`, receive `ACK|SYN` with the expected acknowledgement number
   - Send the final `ACK` and move to `ESTABLISHED`

3. Loss robustness:
   - If handshake packets are lost, retransmission/re-processing should allow the connection to complete eventually (as required by the test programs)

### 3.4 Reliable Data Transfer (RDT within ESTABLISHED)
Inside the `ESTABLISHED` state, implement reliable in-order delivery for application payload:

1. Receiver logic:
   - Maintain an `expect_seq` (next expected sequence number)
   - If an incoming data segment matches `expect_seq`, accept it, append payload into the receive buffer, and advance `expect_seq`
   - If out-of-order data arrives, discard it and send an `ACK` for the expected sequence number (or an equivalent mechanism)

2. Sender logic:
   - Maintain `base` and `nextseq`-like pointers over a send buffer
   - Send segments in the allowed window range
   - On receiving an acknowledgement that advances `base`, remove acknowledged segments from outstanding tracking

3. Retransmission:
   - Implement timeout-driven retransmission for unacknowledged segments using an internal timer mechanism
   - Optionally implement fast retransmit triggered by duplicate ACKs (if required by the assignment/test; present in this project's code path)

### 3.5 Sliding Window Transmission
Implement a sender-side sliding window mechanism that coordinates:

1. Window state
   - send buffer management (store outgoing payload bytes)
   - `base` (oldest unacknowledged sequence)
   - `nextseq` (next sequence to assign/schedule for sending)

2. Window constraints
   - flow-control constraint: do not exceed the receiver’s advertised window
   - congestion-control constraint: do not exceed `cwnd` when congestion control is enabled

3. Multi-threaded/timer integration (implementation-dependent)
   - Maintain a sending loop (e.g., a send thread) that periodically schedules transmissions
   - Maintain a timer queue that triggers retransmissions on timeout

### 3.6 Flow Control (Receiver Advertised Window)
Implement receiver-advertised flow control:

1. Receiver window (`rwnd`)
   - Compute the remaining space in the receiver buffer
   - Advertise this remaining capacity in the `advertised_window` field of ACK packets

2. Sender window adjustment
   - Update the sender’s effective send allowance (`rwnd`) based on the latest advertised value
   - Ensure the sender does not send beyond what the receiver can buffer

### 3.7 Congestion Control (cwnd / ssthresh)
Implement congestion control logic (enabled in this codebase using `CONGESTION_CONTROL`):

1. Congestion states
   - Slow Start (`SLOW_START`)
   - Congestion Avoidance (`CONGESTION_AVOIDANCE`)
   - Fast Recovery (`FAST_RECOVERY`)
   - Congestion Timeout (`CONGESTION_TIMEOUT`)

2. Update rules
   - Increase `cwnd` on acknowledgements depending on the congestion state
   - Update `ssthresh` and `cwnd` on timeout and recovery events

3. RTT estimation and RTO update
   - Estimate RTT from measured sample RTTs for acknowledged segments
   - Update deviation and compute a new RTO (timeout interval) used by the retransmission timers

### 3.8 Connection Teardown (Four-Way FIN Teardown)
Implement FIN-based teardown behavior:

1. FIN handshake
   - Sender initiates closure by sending `FIN` (or a combined FIN/ACK packet, depending on the design)
   - Receiver responds appropriately (e.g., `FIN_ACK` / `ACK` transitions)

2. State transitions
   - Support states such as `FIN_WAIT_1`, `FIN_WAIT_2`, `CLOSE_WAIT`, `LAST_ACK`, and `TIME_WAIT`

3. Loss handling
   - Ensure teardown completes even when FIN/ACK packets are lost (using retransmission/timeouts as needed)

## 4. Validation / Evaluation Plan (No Pre-made Report Included)
To validate that the implementation meets the functional requirements, you should run the provided test programs:

1. Connection establishment test
   - Build and run the client/server executables using the provided `Makefile`
   - Verify the protocol reaches `ESTABLISHED`

2. Reliable data transfer test
   - Use the RDT-like client/server tests that send a large number of segments
   - Verify correct ordering and that data is received without duplication/omission

3. Congestion/network-condition test (if supported by your environment)
   - Use the python automation (`test_congestion.py`) and/or compile the `rdt_client`/`rdt_server` binaries
   - Verify the system continues to make progress under configurable delay/loss/bandwidth conditions

4. Connection teardown test
   - Run close tests that exercise sequential and simultaneous shutdown scenarios
   - Verify the socket eventually reaches `CLOSED`

