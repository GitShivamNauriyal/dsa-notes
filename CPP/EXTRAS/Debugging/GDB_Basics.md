---
tags: [cpp, extras, debugging, gdb, core-dumps]
links: ["[[../../00_Home]]"]
---

# Extras -- GDB Basics

*<- [[../../00_Home|Home]]*

---

GDB (GNU Debugger) is the standard tool for debugging C++ applications on Unix-like systems.

---

## 1. Core Command Cheatsheet

| Command | Shorthand | Description |
|---|---|---|
| `run` | `r` | Start program execution |
| `break <location>` | `b` | Set a breakpoint (e.g. `b main`, `b src/main.cpp:42`) |
| `next` | `n` | Step over the next line of code |
| `step` | `s` | Step into the next line of code |
| `continue` | `c` | Resume program execution until the next breakpoint |
| `print <expr>` | `p` | Print the value of a variable or expression |
| `backtrace` | `bt` | Print the call stack frames up to the current location |
| `info locals` | | Show local variables in the current stack frame |

---

## 2. Watchpoints (Data Breakpoints)

A **Watchpoint** stops program execution whenever the value of a monitored variable or memory address changes. This is extremely useful for tracking down pointer corruption or silent state updates.

- **`watch <var>`**: Suspend execution when `var` is modified (Write Watchpoint).
- **`rwatch <var>`**: Suspend execution when `var` is read (Read Watchpoint).
- **`awatch <var>`**: Suspend execution when `var` is either read or written to.

```bash
(gdb) b main
(gdb) r
(gdb) watch myCounter
Hardware watchpoint 1: myCounter
(gdb) c
Hardware watchpoint 1: myCounter

Old value = 0
New value = 1
0x000000000040114f in updateCounter () at main.cpp:10
```

---

## 3. Core Dumps Inspection

When a C++ program crashes (e.g., due to a Segmentation Fault), the OS can write the state of the program's memory at the moment of the crash into a file called a **Core Dump**.

### A. Enabling Core Dumps (Linux)
By default, core dumps are often disabled (size limit set to 0). Enable them in your shell:
```bash
# Allow unlimited core dump file size
ulimit -c unlimited
```

### B. Loading a Core Dump in GDB
To inspect the crash, pass the executable and the generated core file to GDB:
```bash
gdb ./my_app core
```

### C. Finding the Crash Location
Once GDB loads, use the **`backtrace`** command to view the call stack and identify the exact line of code that triggered the crash:
```bash
(gdb) bt
#0  0x0000000000401124 in crashyFunc () at main.cpp:5
#1  0x0000000000401140 in main () at main.cpp:12
(gdb) frame 0
(gdb) info locals
# Prints variables at the moment of crash
```

---

*<- [[../../00_Home|Home]]*
