---
tags: [cpp, extras, compiler-flags, gcc, clang, optimization-levels, warnings]
links: ["[[../../00_Home]]"]
---

# Extras -- Compiler Flags Reference

*<- [[../../00_Home|Home]]*

---

Compiler flags control optimization, warning levels, code generation, and binary features.

---

## 1. Optimization Level Flags (-O)

| Flag | Meaning | Recommended For | Tradeoffs |
|---|---|---|---|
| **`-O0`** | No optimization | Active Debugging | Faster compilation; slow execution; matches source lines exactly in GDB. |
| **`-O1`** | Basic optimizations | Fast Builds | Moderate speedup; preserves basic debuggability. |
| **`-O2`** | Recommended production | Release Builds | Turns on all optimizations that do not increase binary size. |
| **`-O3`** | Aggressive optimizations | Math/SIMD Hot paths | Enables auto-vectorization and loop unrolling; can increase binary size. |
| **`-Ofast`** | Extreme mathematical optimization | Graphics / Non-critical DSP | **Warning**: Enables `-ffast-math` which disregards IEEE-754 standards (banned in financial math!). |

---

## 2. Warning and Safety Flags

Robust projects treat compiler warnings as errors to maintain code quality.

- **`-Wall`**: Enables a broad collection of common warnings (like unused variables or uninitialized variables).
- **`-Wextra`**: Enables additional useful warnings not covered by `-Wall` (like sign-compare or unused parameters).
- **`-Werror`**: Treats all warnings as compile errors, halting compilation immediately if any warning is triggered.
- **`-Wpedantic`**: Issues warnings for code that violates strict ISO C++ standards.

```bash
# Standard compilation command for robust production builds:
g++ -std=c++20 -O3 -Wall -Wextra -Werror main.cpp -o main
```

---

## 3. Link-Time & Architecture Flags

### A. Link-Time Optimization (`-flto`)
Normally, compilers optimize each translation unit (source file) individually. **`-flto`** defers optimization to the linker stage.
- **Benefit**: Enables the compiler to perform global optimizations (like inlining functions defined in different source files) across the entire program.
- **Tradeoff**: Increases link times significantly.

### B. Architecture Target Optimization (`-march`)
By default, compilers generate generic x86 instructions to ensure the binary runs on any CPU.
- **`-march=native`**: Tells the compiler to detect the host CPU architecture and enable all native instruction sets (like AVX2, FMA, SSE4.2).
- **Warning**: The resulting binary **will not run** on older CPUs lacking these hardware features.

---

*<- [[../../00_Home|Home]]*
