# HIT OSLab Environment (linux-0.11 in Bochs) — English Lab Manual

This English document describes how to install, recover, run, and debug the archived HIT OSLab environment contained in:
`hit-oslab-1/`

Important note: you asked for `hit-pslab-1`, but in the current workspace the directory name is `hit-oslab-1` and it contains the OSLab environment scripts. If your intended directory is different, tell me the exact folder name/path and I will regenerate the document accordingly.

---

## 1. What this repository is for

The repository is an archive of the environment/tooling used in HIT OS labs, providing:

1. A one-command style setup script to install dependencies and extract:
   - `linux-0.11` (baseline kernel source)
   - Bochs configuration and the HDD/hdc image
2. Scripts to:
   - mount the hdc image
   - start Bochs (for i386/amd64 variants)
   - restore/reset the extracted `linux-0.11` directory between lab attempts
3. Example/utility programs in `common/files/` (e.g., `testlab2.c`, `process.c`) and a statistic/log helper `stat_log.py`.

---

## 2. Host prerequisites

The repository README states this is intended for Ubuntu 18.04/typical Linux environments (with `sudo` available when required).

Key tools used by `setup.sh`:

- `sudo`, `apt-get`
- `dpkg` for installing `gcc-3.4` packages
- Bochs + SDL packages for 64-bit (amd64) mode
- `bin86`, `gcc-multilib`, build-essential

---

## 3. Installation

`setup.sh` installs the OSLab environment into:
`$HOME/oslab`

The README suggests using:

```bash
git clone https://github.com/DeathKing/hit-oslab.git ~/hit-oslab
cd ~/hit-oslab
./setup.sh
```

### 3.1 Skip update (optional)

To skip apt source update steps, run:

```bash
./setup.sh -s
```

### 3.2 What `setup.sh` does (high-level)

From reading `setup.sh`, the script:

1. Sets `OSLAB_INSTALL_PATH=$HOME/oslab`
2. Creates:
   - `$OSLAB_INSTALL_PATH`
   - `$OSLAB_INSTALL_PATH/linux-0.11`
3. Extracts:
   - `common/linux-0.11.tar.gz` into `$OSLAB_INSTALL_PATH/linux-0.11`
4. Extracts:
   - `common/bochs-and-hdc.tar.gz` into `$OSLAB_INSTALL_PATH/`
5. Copies common helper files:
   - `common/files/` into `$OSLAB_INSTALL_PATH`
6. Detects host bitness (`getconf LONG_BIT`):
   - 64-bit: installs gcc-3.4 amd64 packages + deps for amd64 Bochs
   - 32-bit: installs gcc-3.4 i386 packages + deps for i386

---

## 4. Recover / Reset between lab runs

Because each lab attempt may modify `linux-0.11`, the environment provides a recovery step.

From README:

```bash
# in oslab directory (after setup)
./run init
```

This is implemented in `amd64/run` and `i386/run` by checking for a backup file `linux-0.11.tar.gz` and extracting it back into `linux-0.11/`.

---

## 5. Mount the hdc image

Before running Bochs, you typically need to mount the provided hdc image as a filesystem.

The scripts `amd64/mount-hdc` and `i386/mount-hdc` mount:
`hdc-0.11.img` at:
`$OSLAB_PATH/hdc`

They use:
- `sudo mount -t minix -o loop,offset=1024 ...`

So the typical flow is:

1. `./mount-hdc` (inside `amd64/` or `i386/` directory depending on architecture)
2. start Bochs with `./run`

The repository also includes logic in `i386/rungdb` to unmount `hdc` first if it is mounted at an unexpected state.

---

## 6. Run the OS in Bochs

### 6.1 amd64 run

Script: `amd64/run`

It performs recovery if `./run init` is passed, unmounts `hdc` if needed, then runs:
`bochs -q -f $OSLAB_PATH/bochs/bochsrc.bxrc`

### 6.2 i386 run

Script: `i386/run`

Similar recovery/unmount logic, but starts Bochs in a GDB-enabled configuration:
`$OSLAB_PATH/bochs/bochs-gdb -q -f $OSLAB_PATH/bochs/bochsrc-gdb.bxrc`

---

## 7. Debugging with GDB

The repository provides multiple debugging wrappers in `i386/`:

### 7.1 Start Bochs with debug graphics

- Script: `dbg-asm`
- Script: `dbg-c`

Both invoke Bochs debug modes using the appropriate bxrc config:
- `bochs-dbg -q -f $OSLAB_PATH/bochs/bochsrc.bxrc`

### 7.2 Use command-driven GDB session

- Script: `rungdb`

It runs:

`$OSLAB_PATH/gdb -x $OSLAB_PATH/gdb-cmd.txt $OSLAB_PATH/linux-0.11/tools/system`

The provided `gdb-cmd.txt` sets:
- `break main`
- `target remote localhost:1234`
- `continue`

This indicates the Bochs GDB server is listening on `localhost:1234` and GDB should connect and continue to `main`.

---

## 8. Example programs and how they relate to labs

The repository provides a couple of example C files in:
`common/files/`

### 8.1 `testlab2.c`: whoami / iam syscall test harness

`common/files/testlab2.c` is a small user program that tests two system calls:

- `iam(char *name, unsigned int size)` via `_syscall1(int, iam, const char* name)`
- `whoami(char *name_buf, unsigned int size)` via `_syscall2(int, whoami, char* name, unsigned int size)`

The program contains a set of test cases (`TEST_CASE`) and prints PASS/ERROR based on:

1. Return value from `iam()`
2. Return value and content written by `whoami()`

At the top it states:
`Compile: "gcc testlab2.c"`
`Run: "./a.out"`

In practice (for a full lab), you typically need:
- system call stubs enabled in the kernel/user environment
- the OS image (hdc) to contain/expose the compiled program

### 8.2 `process.c`: CPU/I/O bound simulation helper

`common/files/process.c` provides:
- a function `cpuio_bound(last, cpu_time, io_time)`

It repeatedly:
- uses `times()` to busy-wait for CPU time
- uses `sleep(1)` loops to simulate I/O time

This is typically used in scheduling/lab experiments where processes alternate CPU and I/O bursts.

### 8.3 `stat_log.py`: scheduling log/statistics helper

`common/files/stat_log.py` provides a CLI:

- input: `/path/to/process.log`
- optional include/exclude PID lists
- options:
  - `-x PID1 PID2 ...` to exclude
  - `-m` and `-g` to control output and “graphic” printing

The top part of the script defines a “cool graphic” legend where states are mapped to symbols:
- `-` new/exit
- `#` running
- `|` ready
- `:` waiting

Use this to transform an experiment-generated process log into summary statistics such as average turnaround/waiting and a timeline graphic.

---

## 9. Suggested lab workflow

Since this repository is an environment archive, a practical workflow is:

1. Run `./setup.sh` on the host.
2. Mount `hdc` (amd64/i386) via `./mount-hdc`.
3. If needed, restore `linux-0.11` using `./run init`.
4. Start Bochs with `./run`.
5. Use `rungdb` or `dbg-*` wrappers to debug kernel/user code with GDB.
6. Rebuild your modified kernel and user programs.
7. Run user test programs (like `testlab2.c`) inside the Bochs environment.
8. Collect logs and optionally process them with `stat_log.py`.

---

## 10. What to include in your English lab submission (suggested)

If you need a write-up, include:

- Installation steps (`setup.sh` usage, where it installs)
- Recovery steps (`./run init`)
- How you start Bochs (amd64 vs i386)
- How you debug (rungdb / dbg-c / dbg-asm)
- What you tested (e.g., `testlab2.c` syscall behavior; process scheduling via `process.c`)
- Optional graphs/statistics derived from `stat_log.py`

---

End of document.

