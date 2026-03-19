# TJU OS Experiment — Discretized Unix V6++ Memory Management (Lab 01)

This English lab manual is generated from the materials under `tju-os-experiment/` and the corresponding modified Unix V6++ code under `tju-os-experiment/oos/`.
It focuses on:

- Migrating from contiguous physical memory management to **page-based discrete physical memory management**
- Implementing a **bitmap-based page-frame allocator**
- Simplifying the two-level paging mechanism to use **per-process private user page tables**
- Adding **Copy-On-Write (COW)** using the **Page Fault** mechanism

Primary references:
`note/pre.md`, `note/lab01.md`, `note/cow.md`, `note/lab01_solution_v2.md`, and key implementation files in `oos/src/`.

---

## 1. Lab Objectives

After completing the lab, the system should support the following high-level behaviors:

1. **Discrete (page-based) physical memory allocation**
   - Manage physical page frames with a **bitmap** rather than a continuous free list / map.
2. **Paging based on a two-level page table**
   - Remove unnecessary relative virtual-to-physical mapping table(s) used in the contiguous approach.
   - Keep the virtual memory layout, while updating the mapping logic to work with non-contiguous physical page frames.
3. **Per-process user page table model**
   - Share the 0# user page table if applicable, while keeping the 1# user page table private per process.
4. **No Disk Swap Area (Design Constraint)**
   - The lab requirement states that the discrete page-based storage design should avoid relying on the on-disk swap area for user process image persistence.
   - Practically, the implementation flow should ensure program execution after `fork/exec` can proceed by using page-frame allocation + page table mapping, and COW for efficient sharing.
5. **Copy-On-Write (COW)**
   - After `fork`, parent and child share user physical page frames as read-only.
   - On the first write to a shared page, trigger a **Page Fault**.
   - Handle the fault by allocating a new page frame and copying contents (only for the faulting page).

---

## 2. System Background and Key Concepts

### 2.1 Physical memory and page frames

From `note/pre.md`, the Bochs VM simulates about **32MB** of physical memory.
The lab targets a page-frame size of `0x1000` (4KB).

The bitmap is sized to manage up to `BITMAP_MAX_SIZE = 0x2000` page frames, using:

- `BITMAP_MAX_SIZE / 8` bytes of storage for the bitmap array
- One bit per physical page frame (`0` = free, `1` = used)

### 2.2 Two-level paging (high-level)

Unix V6++ uses a two-level paging mechanism:

- A page directory points to page tables (including user tables)
- Translation from user virtual addresses to physical addresses relies on these tables

The contiguous-memory design could rely on relative virtual-to-physical structure, but after discretization physical page frames are no longer guaranteed to be contiguous.
Therefore, the mapping logic must be replaced by explicitly filling page table entries for the relevant address ranges.

---

## 3. Repository Structure (What to Look At)

Key documentation:

- `note/pre.md`: motivation, memory model, and experiment plan
- `note/lab01.md`: step-by-step Lab 01 solution (discretization without COW focus)
- `note/cow.md`: COW (copy-on-write) mechanism design
- `note/lab01_solution_v2.md`: detailed lab write-up, including implementation rationale and step flow

Key implementation files:

- `oos/src/include/Bitmap.h`
- `oos/src/include/PageManager.h`, `oos/src/mm/PageManager.cpp`
- `oos/src/include/MemoryDescriptor.h`, `oos/src/proc/MemoryDescriptor.cpp`
- `oos/src/include/ProcessManager.h`, `oos/src/proc/ProcessManager.cpp`
- `oos/src/interrupt/Exception.cpp` (Page Fault handler and COW write handling)
- `oos/src/include/Text.h`, `oos/src/proc/Text.cpp` (shared text segment refcount)

---

## 4. Implementation Plan (Step-by-Step)

### Step 1. Add bitmap-based physical page-frame management

#### 4.1 Bitmap data structure

In `oos/src/include/Bitmap.h`, the bitmap-related constants and structure are defined:

- `BITMAP_MAX_SIZE = 0x2000`
- `struct Bitmap { unsigned char m_bitmap[BITMAP_MAX_SIZE / 8]; unsigned long m_size; }`

#### 4.2 PageManager changes

In `oos/src/mm/PageManager.cpp`, `PageManager` implements:

- `Initialize()`:
  - Clear the bitmap array
- `AllocMemory(size, physicalPageFrameAddr)`:
  - Scan bitmap bits from 0 to `bitmap.m_size`
  - Mark selected bits as used
  - Write allocated physical page frame start addresses into `physicalPageFrameAddr`
  - If it cannot allocate the full request, it frees partially allocated frames
- `FreeMemory(size, physicalPageFrameAddr)`:
  - Clear corresponding bitmap bits

Two derived managers exist:

- `KernelPageManager`:
  - Uses a bitmap size derived from `KERNEL_PAGE_POOL_SIZE`
  - Allocates within kernel pool address range
- `UserPageManager`:
  - Uses a bitmap size derived from `USER_PAGE_POOL_SIZE`
  - Adds reference count tracking for COW support

#### 4.3 Reference counting storage (for COW)

In `oos/src/include/PageManager.h` and `oos/src/mm/PageManager.cpp`, `UserPageManager` maintains:

- `unsigned char m_sharedProcessCnt[0x2000];`

It provides:

- `incrementReferenceCount(physicalPageFrameAddr)`
- `decrementReferenceCount(physicalPageFrameAddr)` (returns decremented count)
- `getReferenceCount(physicalPageFrameAddr)`

---

### Step 2. Modify paging to remove the contiguous-relative mapping dependence

#### 4.4 Per-process 1# user page table and Swtch simplification

Based on `note/lab01.md`, the paging modification key idea is:

- Remove the relative virtual-to-physical mapping table
- Share the 0# user page table across processes if possible
- Keep the 1# user page table private per process
- During context switch, only update the page directory entry that points to the current process's 1# user page table base

This is reflected in `oos/src/proc/ProcessManager.cpp`:

- `ProcessManager::Swtch()` updates the page directory entry:
  - `pd.m_Entrys[1].m_PageTableBaseAddress = (child->...m_UserPageTableArray1 - KERNEL_SPACE_START_ADDR) / PAGE_SIZE;`
  - then flushes the page directory via `FlushPageDirectory()`

#### 4.5 Mapping logic in MemoryDescriptor

In `oos/src/proc/MemoryDescriptor.cpp`, `MemoryDescriptor::MapToPageTable()` builds the private 1# user page table mappings.

Key behavior:

- If text structure is not allocated (`p_textp == NULL`), it returns early (mapping is not ready).
- It clears the present flag in the 0# user page table entries that must be absent (as required by the design).
- It computes page table index ranges for:
  - Text / code region
  - Data region
  - Stack region
- For each 1# user page table entry:
  - Set `m_Present = 1` for valid pages
  - For text pages, set `m_ReadWriter = false`
  - For data and stack pages, set `m_ReadWriter` based on the COW reference count:
    - if reference count is `1`, allow writes
    - otherwise, keep read-only to trigger Page Fault on write

This aligns with the COW mechanism described in `note/cow.md`.

---

### Step 3. Full discretization and process lifecycle integration

This step integrates memory allocation and page table establishment into:

- System initialization (`SetupProcessZero`)
- Process creation (`NewProc`, used by `fork`)
- Program replacement (`Exec`)

#### 4.6 SetupProcessZero

In `oos/src/proc/ProcessManager.cpp`, `SetupProcessZero()`:

- Initializes process 0 control block fields
- Sets `p_pageFrameAddr[0]` to the fixed PPDA address
- Sets `u.u_MemoryDescriptor.m_UserPageTableArray1 = (PageTable*)0xc0203000`
  - This matches the reserved 1# user page table referenced in `note/lab01.md`

#### 4.7 NewProc (fork path) — allocate minimal private state, share page frames

In `ProcessManager::NewProc()`:

- A new process entry is allocated by cloning the current process structure (`current->Clone(*child)`).
- The child gets a new physical page frame for `p_addr` (PPDA/core stack region):
  - `upm.AllocMemory(PageManager::PAGE_SIZE, &desAddress)`
- For the other pages:
  - the child copies references to the parent's page frame addresses
  - it increments reference counts:
    - `upm.incrementReferenceCount(current->p_pageFrameAddr[i])`
- It also marks data and stack pages as **read-only** in the child page table:
  - `pgTable->m_Entrys[i].m_ReadWriter = false;`

Finally:

- It copies a prepared page table (`pgTable`) into the child's private 1# user page table array:
  - `Utility::MemCopy(pgTable, u.u_MemoryDescriptor.m_UserPageTableArray1, PageManager::PAGE_SIZE)`

#### 4.8 Exec — load a new program and rebuild mappings

In `ProcessManager::Exec()`:

1. It parses PE header via `PEParser parser`:
   - sets:
     - `m_TextStartAddress`, `m_TextSize`
     - `m_DataStartAddress`, `m_DataSize`
     - `m_StackSize`
2. It frees old shared text and reduces mappings via reference counting:
   - `u.u_procp->p_textp->XFree()` and `userPgMgr.decrementReferenceCount(...)`
3. It allocates a new shared `Text` or reuses existing one:
   - allocates memory for the text segment via `userPgMgr.AllocMemory(pText->x_size, pText->x_physicalAddr)`
4. It expands the process image by pages:
   - `u.u_procp->Expand(newSize)`
5. It initializes the new process's 1# user page table:
   - copies template from the reserved user page table base:
     - `srcAddr = (unsigned long)Machine::Instance().GetUserPageTableArray() + 0x1000;`
     - `desAddr = (unsigned long)u.u_MemoryDescriptor.m_UserPageTableArray1;`
   - calls `u.u_MemoryDescriptor.MapToPageTable();`
6. It loads program segments into memory:
   - `parser.Relocate(pInode, sharedText);`

---

### Step 4. Implement Copy-On-Write (COW) via Page Fault

The lab uses write-triggered faults to implement the “copy on first write” policy.

#### 4.9 What triggers COW?

After `fork`:

- parent and child share physical page frames
- both processes see those pages as **read-only**

When either process tries to write to a shared page:

- the CPU raises a Page Fault due to write permission violation
- the kernel Page Fault handler detects that the faulting address belongs to a COW-protected region
- it allocates a new page frame, copies contents, updates mapping entries, and sets the page back to writable

#### 4.10 Page Fault handler (COW write path)

In `oos/src/interrupt/Exception.cpp`, `Exception::PageFault(...)` handles:

- Read `cr2` to get the faulting linear address
- If in user mode:
  - Check whether `cr2` is within data region or stack region
  - Compute:
    - `pageFrameAddr` (virtual-index based address for the faulted page)
    - `pageFrameIdx` (page index)
    - map from the corresponding 1# user page table entry to the physical page frame address:
      - `physicalPageFrameAddr = ...m_PageBaseAddress * PAGE_SIZE`
  - Then apply COW logic:
    - if `decrementReferenceCount(physicalPageFrameAddr)` becomes `0` or otherwise indicates sharing handling:
      - allocate a new physical page frame via `upm.AllocMemory(PageManager::PAGE_SIZE, &newAddr)`
      - update `current->p_pageFrameAddr[idx]` for that page
      - update `md.m_UserPageTableArray1->m_Entrys[...] .m_PageBaseAddress`
      - copy the page content:
        - `Utility::CopySeg(src + i, des + i)` in a loop of bytes
    - finally set the page entry's `m_ReadWriter = true`
    - flush page directory to ensure new translation is used:
      - `FlushPageDirectory();`
  - Else if fault suggests stack growth condition:
    - call `current->SStack()`
  - Otherwise treat as invalid access (SIGSEGV)

#### 4.11 Shared text segment reference counting

For `exec` and `exit` correctness, the shared text segment is also reference-counted.

In `oos/src/proc/Text.cpp`:

- `Text::XccDec()` decrements `x_ccount`, and when it reaches `0` it frees the physical memory via `UserPageManager::FreeMemory(x_size, x_physicalAddr)`

This ensures text segment memory is released only when the last process using it exits or replaces its address space.

---

## 5. How to Debug and Validate (Suggested Workflow)

Your repository provides a debugging workflow in `note/lab01.md`:

1. Open `oos` as the root folder in VSCode
2. Use `oos/.vscode/launch.json`:
   - `Debug Kernel` is typically sufficient for this lab
3. Use VSCode debug console with GDB:
   - `bt` / `backtrace`
   - `x/<n><format><size> <address>`
   - `p/<format> <var>` to inspect values
   - `step/s`, `next/n`, and breakpoints

Validation strategy aligned with the design:

- **Initialization checks**
  - confirm bitmap initialization
  - confirm 1# user page table pointer exists for new processes
- **fork behavior**
  - after `fork`, confirm data/stack pages are marked read-only in page tables
  - confirm page frame reference counts are incremented
- **write + Page Fault**
  - trigger writes to data/stack regions in parent and child separately
  - verify each write causes a Page Fault and results in a private copy for that page
- **exec/exit correctness**
  - verify that replacing a program frees old pages correctly via reference counting
  - verify system does not leak pages or free pages prematurely

---

## 6. Summary of Deliverables (What You Implemented)

This lab manual corresponds to the following key engineering deliverables:

- Bitmap-based page-frame allocation:
  - implement and use bitmap in `PageManager`
- Two-level paging update:
  - per-process private 1# user page table
  - simplify `Swtch` by updating PDE base address only
  - implement `MemoryDescriptor::MapToPageTable()` to explicitly fill PTEs
- COW mechanism:
  - add reference counting in `UserPageManager`
  - mark shared pages as read-only after `fork`
  - implement Page Fault-based write handling in `Exception::PageFault()`

---

End of document.

