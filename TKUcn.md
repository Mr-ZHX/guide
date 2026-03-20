# Network Experiments (Computer Network Labs): English Experiment Descriptions (Total)

This document consolidates the lab requirements and functional tasks from the `netweork` directory (Network course labs). It focuses on what you are required to build/verify and what to implement, and intentionally avoids including any “completed report” results/answers.

---

## Experiment 1: Socket Programming (Two-Person Chat Protocol)

### Experimental Requirements
1. Using stream sockets, design a two-person chat protocol. The chat messages must contain a time label. You must fully describe the interaction message handling details, including message types, syntax, semantics, and timing/sequence.
2. Design the chat program, including:
   - module decomposition,
   - module functions,
   - a module flow chart.
3. On Windows, implement the designed program using C/C++.
   - A command-line interface is acceptable.
   - While coding, you may only use the basic socket functions.
   - It is not allowed to use socket wrapper classes/architectures.
4. Test the implemented program.
5. Write the experiment report and submit the report and the source code.

### Functions (What the Program Should Do)
- Implement two-person chat communication.
- Allow sending and receiving messages at any time.
- Use TCP with stream sockets.
- Provide a normal exit/termination method.

### What Needs to Be Implemented
- Define and implement the required chat protocol, including the time label and the complete message handling specification (types/syntax/semantics/timing).
- Architect the client/server (or peer) modules with clear responsibilities and provide module flow diagrams.
- Implement the chat client and server on Windows in C/C++ using only basic socket APIs.
- Implement message send/receive logic such that communication can proceed at arbitrary times.
- Implement proper shutdown/exit behavior and verify it with tests.

### Optional / Bonus (If Required by Your Course Rubric)
- Multi-person chat.
- Multi-threaded programming to continuously send/receive messages.
- Use line-feed truncation to send spaces/tabs (or equivalent framing mechanism).
- Support both English and Chinese messages.
- Support broadcast/group-send.

---

## Experiment 2: Web Server Setup + Wireshark Packet Capture

### Experimental Requirements
1. Set up a Web server (you may choose any OS) and create a simple Web page containing:
   - simple text information (at least: major, student ID, name),
   - your own LOGO.
2. Use a browser to access your Web page, capture the browser-to-Web-server interaction using Wireshark, and provide a brief analysis.
3. Submit the experiment report.

### Functions (What You Must Achieve)
- A working Web server serving your created Web page.
- A Wireshark capture of the browser accessing your page, covering the interaction process.
- A brief written analysis of the captured interaction process.

### What Needs to Be Implemented
- Create the HTML page with the required text fields and logo, then place it in your Web server’s document root.
- Start the Web server and access the page through a browser.
- Capture the traffic with Wireshark and select key packets/frames relevant to your brief analysis.
- Write and submit the experiment report describing your capture and analysis.

---

## Experiment 3-1: Reliable Connection-Oriented Data Transfer over UDP (Stop-and-Wait)

### Experimental Requirements
Implement a connection-oriented reliable data transfer in user space using datagram sockets, with the following functions:
1. Connection establishment
2. Error detection
3. Confirmed retransmission (ACK-based retransmission)
4. Stop-and-wait mechanism
5. Connection termination/disconnection

### Functions (Protocol/Behavioral Capabilities)
- Provide a “connection-like” reliable transfer service on top of UDP.
- Detect corrupted/erroneous packets.
- Retransmit when confirmations are missing or packets are lost/corrupted.
- Use a stop-and-wait behavior for sending/receiving data.
- Terminate the connection cleanly.

### What Needs to Be Implemented
- A UDP-based sender and receiver that implement:
  - connection establishment,
  - reliable transfer with stop-and-wait,
  - error detection + retransmission upon failure,
  - connection termination.

---

## Experiment 3-2: Reliable Transfer with Sliding Window (SR Protocol)

### Experimental Requirements
Based on Experiment 3-1, implement the sliding window mechanism, including:
1. Add sliding window functionality.
2. Use the SR (Selective Repeat) protocol.
3. Use multi-threaded programming.
4. Be able to scan files in a folder to support file transfer and verification conveniently.

### Functions (What You Must Extend from 3-1)
- Extend reliable transfer from stop-and-wait to sliding window.
- Implement SR sender/receiver behavior (selective retransmission within a receiver window).
- Support practical testing by scanning a folder for files to send/verify.

### What Needs to Be Implemented
- Modify the sender/receiver logic to support a sliding window using SR.
- Ensure retransmissions are handled selectively according to SR semantics.
- Add multi-threaded logic as required by the lab (e.g., decouple sending/receiving).
- Implement folder scanning to enumerate available files for transfer and verification.

---

## Experiment 3-3: Add Congestion Control (Reno + Flow Control)

### Experimental Requirements
Based on Experiment 3-2, implement congestion control with:
1. Add flow control functionality.
2. Use the Reno algorithm.
3. Use multi-threaded programming.
4. Be able to scan files in a folder to support file transfer and verification conveniently.

### Functions (What You Must Extend from 3-2)
- Integrate congestion/flow control into the reliable sliding-window protocol.
- Apply Reno behavior to adjust sending rate/window in response to network conditions (e.g., loss signals).
- Maintain reliable transfer while applying congestion control.

### What Needs to Be Implemented
- Extend the protocol to incorporate flow/congestion control using Reno.
- Update sending behavior according to Reno rules so that performance changes under loss/delay are observable.
- Keep the multi-threaded structure and file transfer/verification support via folder scanning.

---

## Experiment 3-4: Performance Comparison Experiments (Delay/Loss Controlled)

### Experimental Requirements
Using the given test environment, by changing delay time and packet loss rate, complete the following three performance comparison experiments:
1. Compare stop-and-wait mechanism vs sliding window mechanism.
2. Evaluate how different window sizes affect performance under sliding window.
3. Compare performance with congestion control vs without congestion control.

### Functions (What Needs to Be Measured/Compared)
- Run controlled experiments under varying delay and loss conditions.
- Collect performance indicators and compare results across the three scenarios:
  - stop-and-wait vs sliding window,
  - different window sizes in sliding window,
  - congestion control enabled vs disabled.

### What Needs to Be Implemented
- Provide an experiment harness/workflow to run tests under multiple delay/loss configurations.
- Ensure you can switch between the compared mechanisms/parameter settings:
  - stop-and-wait vs sliding window,
  - multiple sliding window sizes,
  - congestion control enabled vs disabled.
- Organize the measured outcomes in a way that supports your comparison/report.

