# uestc-operation-system: English Experiment Descriptions (Total)

This document summarizes the lab requirements and functional tasks from `uestc-operation-system`, based on the provided source code and scripts. It is written in English and focuses on what you must build/verify. It intentionally does not include any “completed report” style results or answers.

The repository is intended to run in a Linux environment.

---

## Experiment 1: Dining Philosophers (Multithreading Coordination)

### Experimental Requirements
- Create multiple philosopher threads and simulate dining behavior concurrently.
- Use synchronization primitives (semaphores) to coordinate access to shared resources (chopsticks).
- Ensure the solution avoids deadlock under concurrency.
- Maintain and protect shared “eating status” information so printing output is consistent.

### Functions (Modules to Understand/Use)
- `philosopher.h`
  - Constants: `PHILOSOPHER_NUM`, `CHOPSTICK_NUM`, `ALLOWED_NUM`
  - Shared argument structure: `philosopher_arg`
- `philosopher.c`
  - `run(philosopher_arg *arg)`: philosopher thread routine
  - `eat(philosopher_arg *arg)`: simulated eating/thinking section
  - `display_eating_philosophers(philosopher_arg *arg)`: print current eating set
- `main.c`
  - Thread creation and semaphore initialization

### What Needs to Be Implemented
- Initialize semaphores:
  - One “allowed number” semaphore (`ALLOWED_NUM`) to restrict how many philosophers can try to eat concurrently.
  - A semaphore for each chopstick (`CHOPSTICK_NUM`), initialized to 1.
  - A mutex semaphore to protect shared status printing/updates.
- Implement the philosopher thread routine (`run`):
  - Wait on `allowed_num` before picking chopsticks.
  - Acquire left and right chopstick semaphores in a consistent way.
  - Call the eating routine, then release both chopstick semaphores.
  - Post back to `allowed_num`.
- Implement `eat`:
  - Mark the philosopher as “eating” under the mutex.
  - Print which philosophers are currently eating.
  - Simulate time delay.
  - Mark the philosopher as “thinking” again under the mutex and print updated status.
- In `main`, create the required number of threads using `pthread_create` and keep the process alive (e.g., by yielding in a loop).

---

## Experiment 2: Producer–Consumer Problem (Bounded Buffer with Semaphores)

### Experimental Requirements
- Implement a bounded buffer shared by multiple producer and consumer threads.
- Coordinate producers and consumers using semaphores to prevent:
  - Buffer overflow (writing when the buffer is full)
  - Buffer underflow (reading when the buffer is empty)
- Support graceful termination behavior when the input data is exhausted (e.g., on EOF).
- Use multithreading (`pthread`) and synchronization (`semaphore`).

### Functions (Modules to Understand/Use)
- `buffer_pool.h` / `buffer_pool.c`
  - `init_pool(struct buffer_pool *pool, int size)`
  - `put(struct buffer_pool *pool, char c)`
  - `get(struct buffer_pool *pool)`
  - Shared semaphores: `empty`, `full`, `mutex`
- `producer.h` / `producer.c`
  - `run_produce(struct buffer_pool *pool)`
  - `produce(struct buffer_pool *pool)` (typically reads an input byte and inserts into the buffer)
- `consumer.h` / `consumer.c`
  - `run_consume(struct buffer_pool *pool)`
  - `consume(struct buffer_pool *pool)` (removes an item and prints)
- `main.c`
  - Initializes the buffer and starts producer/consumer threads

### What Needs to Be Implemented
- Implement/complete the bounded buffer logic in `buffer_pool`:
  - Provide storage (an in-memory char array).
  - Provide `put` and `get` operations that manipulate buffer indices safely according to the chosen simple design.
  - Initialize semaphores:
    - `empty` to the buffer capacity
    - `full` to 0
    - `mutex` to 1
- Implement producer thread routine (`run_produce`):
  - Wait on `empty` before producing.
  - Wait on `mutex` before inserting into the shared buffer.
  - Produce an item (e.g., read next byte from an input file) and call `put`.
  - If input is exhausted, terminate producer threads appropriately.
  - Post to `full` after insertion; release `mutex`.
  - Optionally sleep/yield to simulate production rate.
- Implement consumer thread routine (`run_consume`):
  - Wait on `full` before consuming.
  - Wait on `mutex` before removing from the shared buffer.
  - Call `get` and print the consumed item.
  - If EOF is detected, stop the consumer thread(s) as required.
  - Post to `empty` after consumption; release `mutex`.
  - Optionally sleep/yield to simulate consumption rate.
- In `main`, create the configured number of producer/consumer threads and keep the program running until threads end.

---

## Experiment 3: Pipe and Message Queue (Inter-Process Communication)

### Experimental Requirements
- Demonstrate two IPC mechanisms using separate programs/process flows:
  1. IPC via a Unix pipe.
  2. IPC via System V message queues.
- Use `fork` to create multiple child processes that send messages to a parent process.
- Parent process receives messages and outputs them in a controlled manner.

### Functions (Modules to Understand/Use)
- `os-section3/pipe/main.c`
  - `pipe(fd)` to create a pipe
  - `fork()` to create children
  - `write(fd[1], ...)` from children
  - `read(fd[0], ...)` in parent
  - `waitpid()` for child synchronization
- `os-section3/queue/main.c`
  - `ftok` to generate a key
  - `msgget` to create/get a message queue
  - `msgsnd` in children (with different `type` values)
  - `msgrcv` in parent to receive messages by type

### What Needs to Be Implemented
- Pipe version:
  - Create the pipe.
  - Fork child process 1 and child process 2.
  - Each child writes a distinct message to the pipe (write-end).
  - The parent waits for both children and then reads data from the read-end and prints it.
  - Ensure correct buffer sizing and handle possible read/write failures.
- Message queue version:
  - Create a System V message queue using a key generated by `ftok`.
  - Fork two children; each child sends a message using `msgsnd` with a unique message `type`.
  - The parent receives messages using `msgrcv` with the expected message types and prints the message content.
  - Ensure the message structure sizes and `msgsnd`/`msgrcv` lengths match the protocol you define.

---

## Experiment 4: Detect File Size Changes

### Experimental Requirements
- Implement a shell script that monitors a user-specified file.
- The script must:
  - Ask the user for a filename (or accept input) and verify the file exists.
  - Check the file size repeatedly.
  - Perform **5 consecutive checks**.
  - If the file size changes at any time, restart the consecutive-check counter from 1 using the new size as the baseline.
  - Wait a fixed interval between checks (the provided design uses a 5-second sleep).

### Functions (Modules to Understand/Use)
- `test.sh`
  - Reads input file name
  - Uses `/usr/bin/ls -l` output parsing to extract file size
  - Loops with an index counter and a delay
  - Restarts the counter on size change

### What Needs to Be Implemented
- Prompt for filename and check existence:
  - If the file does not exist, print an error and exit.
  - If it exists, begin monitoring.
- Initialize baseline size (`file_size`) using the file’s current size.
- Loop until 5 consecutive checks are completed:
  - Print current check number, filename, and current size.
  - Compare current size with baseline.
  - If different:
    - Print a “size changed, restart” message.
    - Update baseline to the new size.
    - Reset the counter to 1.
    - Continue to the next check after the restart logic.
  - Sleep for 5 seconds before the next check.
- After reaching 5 stable checks, print completion and exit.

---

## Experiment 5: Detect Whether a User Logged In

### Experimental Requirements
- Implement a shell script to monitor login status of a given username.
- The script must:
  - Accept a username parameter.
  - Repeatedly query current logged-in users.
  - Display:
    - detection count,
    - the list of currently logged-in users,
    - current time.
  - If the given username is detected in the logged-in user list:
    - print that the user is logged in,
    - stop monitoring and exit.
  - Otherwise:
    - print that the user is not logged in,
    - continue monitoring after a fixed delay (the design uses 5 seconds).

### Functions (Modules to Understand/Use)
- `usr_monitor.sh`
  - `show_login_users()` uses `who` to list usernames and prints them
  - `contains()` checks whether the target username is present in the captured list
  - The main loop clears the screen, prints status, and exits on success

### What Needs to Be Implemented
- Parse command-line argument:
  - If no username is provided, print usage and exit.
- Implement `contains(element, array)`:
  - Determine whether the username is present.
  - Return a boolean-like result.
- Implement monitoring loop:
  - Clear screen each iteration.
  - Increment detection index.
  - Obtain current logged-in users using `who`.
  - Print all detected users.
  - If username is present:
    - print a “logged in” message,
    - exit the script.
  - Otherwise:
    - print “not logged in”.
  - Print current timestamp.
  - Sleep 5 seconds and repeat.

---

## Experiment 6: Implement `cp` Command in C

### Experimental Requirements
- Implement a subset of the Unix `cp` command using C.
- Required behaviors:
  1. Copy a single file: `cp <source_file> <target>`
     - If `<target>` is a directory, copy the source file into that directory.
     - If `<target>` is a file path, copy to that exact destination file.
  2. Recursive directory copy: `cp -r <source_dir> <target_dir>`
     - Recursively copy all files/subdirectories from source to destination.
- Handle argument validation and print a usage message for invalid invocation.
- For recursive copy, create destination directories as needed.
- Use buffered file I/O for data transfer.
- For recursion, you may use `fork` and `wait` to parallelize file copies (as shown in the provided implementation design).

### Functions (Modules to Understand/Use)
- `os-section6/main.c`
  - `copy_to_file(dst, src)`: read from source and write to destination
  - `copy_to_dir(dst_dir, src)`: copy a source file into a destination directory
  - `copy_r(dst, src)`: recursive directory copy
  - `main`: argument parsing and dispatch

### What Needs to Be Implemented
- Argument parsing:
  - Support 3 arguments for non-recursive copy.
  - Support 4 arguments with `-r` for recursive copy.
  - Validate whether source is a directory when `-r` is not provided.
  - For `-r`, ensure destination directory exists (or follow the specified policy).
- Implement file copy (`copy_to_file`):
  - Open source in binary read mode.
  - Open/create destination in binary write mode.
  - Copy data using a fixed-size buffer until EOF.
  - Close files and handle errors.
- Implement directory copy without recursion (`copy_to_dir`):
  - Append the source file’s base name to the destination directory path.
  - Call the file copy function.
- Implement recursive directory copy (`copy_r`):
  - Create the corresponding destination directory.
  - Traverse entries using directory APIs (`opendir`, `readdir`).
  - Skip `.` and `..`.
  - For subdirectories: recurse.
  - For regular files: copy them (optionally using `fork` to run copies concurrently).
  - Use `wait` to ensure all child processes complete before exit.

---

