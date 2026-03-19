# FDVIM — Lab Manual (English)

**A Simple Vim-like Text Editor (Data Structures Project)**

This document is a complete English lab manual for **FDVIM**, a minimal Vim-style console editor implemented in C++. It was developed as a Data Structures course project (Fall 2021, Fudan University). The manual describes the project overview, features, structure, build instructions, user guide, and design.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Features](#2-features)
3. [Project Structure](#3-project-structure)
4. [Environment and Build](#4-environment-and-build)
5. [User Guide](#5-user-guide)
6. [Design](#6-design)
7. [Implementation Summary](#7-implementation-summary)

---

## 1. Project Overview

- **Name**: FDVIM  
- **Purpose**: Design a simple Vim-like editor in C++ as a data structures project.  
- **Platform**: Windows (console application; uses Windows API for console I/O).  
- **Language**: C++ (STL: `vector`, `string`, `stack`, `fstream`; smart pointers for polymorphism).

The editor has **three screens**: Welcome, Edit (with Normal and Insert modes), and Goodbye. All interaction is keyboard-driven.

---

## 2. Features

### 2.1 Screens

| Screen       | Description |
|-------------|-------------|
| **Welcome** | Displays a welcome banner and author/contact info. Press any key to enter the Edit screen (Normal mode). |
| **Edit**    | Main editing area: text content above, mode and command line at the bottom. Two sub-modes: Normal and Insert. |
| **Goodbye** | Displays a farewell message. Press any key to exit the program. |

### 2.2 Normal Mode

- **Prompt**: `<Normal>` at the bottom left.
- **Commands**:
  - `:open filename` — Open (or create) a text file under `assets/` and display it.
  - `:w filename` — Save the current text to a file under `assets/` (creates or overwrites).
  - `:q` — Quit the editor (switch to Goodbye screen).
  - `x` — Delete the character under the cursor.
  - `u` — Undo last edit/move.
  - `r` — Redo last undone action.
  - `/pattern` — Search for exact (whole-word) match of `pattern` from the current cursor position (KMP).
  - `h`, `j`, `k`, `l` — Move cursor left, down, up, right (Vim-style).
  - `i` — Switch to Insert mode.

### 2.3 Insert Mode

- **Prompt**: `<Insert>` at the bottom left.
- **Behavior**:
  - `Esc` — Return to Normal mode.
  - Any other printable input — Insert the character at the cursor and move the cursor right.

### 2.4 Limitations (as in original README)

- Window size is fixed; resizing is not supported.
- No line wrap or pagination.
- All edited files are under the `assets/` directory (paths are relative to the executable).

---

## 3. Project Structure

```
fdvim/
├── assets/              # Test/data files (e.g. 1.txt, 2.txt, 3.txt)
│   └── img/             # Images (e.g. run.png, UML.drawio)
├── include/             # Headers
│   ├── Editor/
│   │   └── Editor.h
│   ├── Screen/
│   │   ├── EditScreen.h
│   │   ├── GoodbyeScreen.h
│   │   ├── Screen.h
│   │   └── WelcomeScreen.h
│   ├── Txt/
│   │   ├── InsertTxt.h
│   │   ├── NormalTxt.h
│   │   └── Txt.h
│   └── Util/
│       ├── Cursor.h
│       └── History.h
├── lib/                 # Implementation
│   ├── Editor/
│   │   └── Editor.cpp
│   ├── Screen/
│   │   ├── EditScreen.cpp
│   │   ├── GoodbyeScreen.cpp
│   │   ├── Screen.cpp
│   │   └── WelcomeScreen.cpp
│   ├── Txt/
│   │   ├── InsertTxt.cpp
│   │   ├── NormalTxt.cpp
│   │   └── Txt.cpp
│   └── Util/
│       ├── Cursor.cpp
│       └── History.cpp
├── tools/               # Main program
│   └── fdvim.cpp
├── CMakeLists.txt
├── build.bat            # Build script
├── LICENSE
└── README.md
```

---

## 4. Environment and Build

### 4.1 Requirements

- **OS**: Windows.
- **Terminal**: Command prompt (cmd).
- **Build**: CMake (version 3.20 or above) with **MinGW Makefiles** generator.

### 4.2 Build Steps

1. Open a terminal in the project root (`fdvim/`).
2. Run:
   ```bat
   build.bat
   ```
   This runs:
   ```bat
   cmake -B build -G "MinGW Makefiles" && cmake --build build
   ```
3. The executable is generated at: **`build/tools/fdvim.exe`**.
4. Run by double-clicking `fdvim.exe` or from cmd: `build\tools\fdvim.exe`.

### 4.3 Window Setup (in code)

- The editor sets the console to 100 columns × 30 lines and the window title to "FDVIM" (see `Editor::Editor()`).

---

## 5. User Guide

### 5.1 Welcome Screen

- ASCII-art "FDVIM" banner and author/email/hint text.
- **Action**: Press any key to continue; the next key may be consumed to enter Edit screen (Normal mode).

### 5.2 Edit Screen — Normal Mode

- **Content area**: Lines 0–27 show the current text (one `string` per line).
- **Status line**: Row 28 shows `<Normal> ` and command/feedback.
- **Commands** (type after the prompt as needed):
  - `:open filename` — Open `assets/filename` (e.g. `:open 1.txt`). Creates the file if it does not exist.
  - `:w filename` — Save buffer to `assets/filename`.
  - `:q` — Quit to Goodbye screen.
- **Single-key actions**: `x`, `u`, `r`, `h`, `j`, `k`, `l`, `i`, `/` (then type pattern and Enter).

### 5.3 Edit Screen — Insert Mode

- **Status line**: `<Insert> `.
- Type normally to insert characters at the cursor.
- **Esc** — Switch back to Normal mode (state is preserved).

### 5.4 Goodbye Screen

- Message: "Thanks for Using!"
- **Action**: Press any key to exit the process.

### 5.5 File Paths

- Open/save paths are prefixed with `../../assets/` relative to the run directory (e.g. when running from `build/tools/`, this points to the project `assets/` folder). Ensure you run from the expected working directory so that `assets/` is found.

---

## 6. Design

### 6.1 Architecture Overview

- **Editor**: Holds the current **Screen** (`std::shared_ptr<Screen>`). Main loop: `run()` repeatedly calls `render()` and `handleInput()` on the current screen.
- **Screens**: Abstract base **Screen** with virtual `handleInput()` and `render()`. Concrete screens:
  - **WelcomeScreen** — Welcome UI; on any key, replaces `Editor::screen` with **EditScreen**.
  - **EditScreen** — Holds current **Txt** (`std::shared_ptr<Txt>`); delegates input/render to it.
  - **GoodbyeScreen** — Farewell message; on key, exits the program.
- **Edit modes**: Abstract base **Txt** with virtual `handleInput()` and `render()`. Concrete:
  - **NormalTxt** — Normal mode: commands, motion, delete, undo/redo, search, switch to Insert.
  - **InsertTxt** — Insert mode: insert character, Esc to switch back to Normal.

So: **Editor → Screen (Welcome / Edit / Goodbye)** and **EditScreen → Txt (Normal / Insert)**. Polymorphism is done with `shared_ptr` to base classes.

### 6.2 Data Structures

- **Text storage**: `std::vector<std::string>` — each element is one line.
- **Cursor**: `COORD cur` (Windows) — row `Y`, column `X` in the text grid.
- **Undo/Redo**: Class **History** with four stacks:
  - `undo_txt`, `redo_txt`: `std::stack<std::vector<std::string>>` for text snapshots.
  - `undo_cur`, `redo_cur`: `std::stack<COORD>` for cursor positions.
  - Before each “undoable” action (e.g. move, delete, insert), the current `(txt, cur)` is pushed onto the undo stacks. Undo pops from undo and pushes to redo; redo does the opposite.
- **Cursor utility**: **Cursor** — static helper (singleton-like) for setting/getting console cursor position via Windows API (`GetStdHandle`, `SetConsoleCursorPosition`, `GetConsoleScreenBufferInfo`).

### 6.3 Search (Normal Mode)

- **KMP** is used for whole-string matching:
  - `KMPPreProcess(pattern, next)` builds the failure table.
  - `KMPSearch(pattern, cur)` searches from the current cursor to the end of the buffer; on match, updates `cur` and shows "Found! ..."; if not found, optionally search from the beginning (prompt "y/n").

### 6.4 Class Diagram (Summary)

```
Editor
  └── std::shared_ptr<Screen> screen
        ├── WelcomeScreen
        ├── EditScreen  ── std::shared_ptr<Txt> text
        │                    ├── NormalTxt  (file, KMP, commands)
        │                    └── InsertTxt (insert)
        └── GoodbyeScreen

Txt (abstract)
  ├── vector<string> txt
  ├── COORD cur
  └── History history

History
  ├── stack<vector<string>> undo_txt, redo_txt
  └── stack<COORD> undo_cur, redo_cur

Cursor (static)
  └── setPos(x,y), getPos(x,y)
```

---

## 7. Implementation Summary

### 7.1 Main Entry

- **`tools/fdvim.cpp`**: Creates an `Editor` and calls `vim.run()`. No other logic.

### 7.2 Editor Loop

- **`Editor::run()`**: `while (true) { render(); handleInput(); }`.
- **`Editor::render()` / `handleInput()`**: Delegate to `screen->render()` and `screen->handleInput()`.

### 7.3 Screen and Mode Switching

- **WelcomeScreen::handleInput()**: On any key, set `Editor::screen = std::make_shared<EditScreen>()`.
- **EditScreen**: Delegates to `text->handleInput()` and `text->render()`. Default `text` is `NormalTxt`.
- **NormalTxt**: On `i`, set `EditScreen::text = std::make_shared<InsertTxt>(txt, cur, history)` so state is preserved.
- **InsertTxt**: On `Esc`, set `EditScreen::text = std::make_shared<NormalTxt>(txt, cur, history)`.
- **NormalTxt**: On `:q`, set `Editor::screen = std::make_shared<GoodbyeScreen>()`.
- **GoodbyeScreen::render()**: Shows message and calls `exit(0)` after `system("pause")`.

### 7.4 File I/O

- **NormalTxt** uses `std::fstream file`.  
- **readTxt(filename)**: Open in read mode; if the file does not exist, create it and reopen; read line-by-line into `txt`; set cursor to end of text; call `history.save(txt, cur)`.  
- **writeTxt(filename)**: Open in write mode; write each line (last line without trailing newline).

### 7.5 Key Files Reference

| File | Role |
|------|------|
| `Editor.cpp` | Console init, run loop, delegation to screen. |
| `WelcomeScreen.cpp` | Welcome UI and transition to EditScreen. |
| `EditScreen.cpp` | Delegation to current Txt. |
| `GoodbyeScreen.cpp` | Farewell and exit. |
| `NormalTxt.cpp` | Commands, motion, delete, undo/redo, KMP search, file open/save, switch to Insert. |
| `InsertTxt.cpp` | Character insertion, Esc to Normal. |
| `History.cpp` | save(), undo(), redo(), canUndo(), canRedo(). |
| `Cursor.cpp` | setPos(), getPos() using Windows console API. |

---

## Appendix: Command Quick Reference

| Key / Command | Mode  | Action |
|---------------|-------|--------|
| Any key       | Welcome | Go to Edit (Normal). |
| `:open fn`    | Normal | Open/create file `fn` in `assets/`. |
| `:w fn`       | Normal | Save to `fn` in `assets/`. |
| `:q`          | Normal | Quit to Goodbye. |
| `x`           | Normal | Delete char at cursor. |
| `u` / `r`     | Normal | Undo / Redo. |
| `/pattern`    | Normal | Search (KMP) from cursor. |
| `h` `j` `k` `l` | Normal | Left, Down, Up, Right. |
| `i`           | Normal | Enter Insert mode. |
| `Esc`         | Insert | Enter Normal mode. |
| (other)       | Insert | Insert character. |
| Any key       | Goodbye | Exit program. |

---

*This lab manual is an English summary of the FDVIM project and its original Chinese README. For the full UML and screenshots, see `assets/img/` and the original README.md.*
