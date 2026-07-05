---
tags: [cpp, performance, benchmarking, profiling, dead-code-elimination]
links: ["[[09_Perf_Index]]", "[[09_Perf_SIMD_Basics]]", "[[09_Perf_Memory_Alignment_and_Padding]]"]
---

# Performance & Optimization -- Benchmarking & Profiling

*<- [[09_Perf_SIMD_Basics|SIMD Basics]] · [[09_Perf_Memory_Alignment_and_Padding|Memory Alignment & Padding ->]]*

---

Microbenchmarking is the practice of timing small code segments. However, optimizing compilers make writing correct benchmarks challenging because they can optimize away the operations you want to measure.

---

## 1. The Dead-Code Elimination (DCE) Trap

When compilers compile code with optimizations enabled (e.g. `-O2` or `-O3`), they perform **Dead-Code Elimination**.
- If a loop performs calculations but the result is **never used** outside the loop, the compiler flags the calculations as dead code.
- The compiler will **delete the entire loop**, resulting in a benchmark running in `0 nanoseconds`!

### The Buggy Benchmark:
```cpp
#include <chrono>
#include <iostream>

void buggyBenchmark() {
    auto start = std::chrono::high_resolution_clock::now();

    // Loop we want to measure
    for (int i = 0; i < 100'000'000; ++i) {
        int x = i * i; // BUG: x is never used! Compiler deletes this entire loop!
    }

    auto end = std::chrono::high_resolution_clock::now();
    std::chrono::duration<double, std::milli> duration = end - start;
    
    // Outputs 0ms because the loop was optimized away!
    std::cout << "Duration: " << duration.count() << " ms\n"; 
}
```

---

## 2. Preventing Compiler Optimization Traps

To get accurate benchmark timings, you must force the compiler to believe the calculated values are utilized.

### Method A: Use volatile sink (Portable)
Assign the final result to a volatile global variable, preventing the compiler from discarding the calculations.

```cpp
volatile int sink = 0;

void benchmarkWithVolatile() {
    auto start = std::chrono::high_resolution_clock::now();

    int temp = 0;
    for (int i = 0; i < 100'000'000; ++i) {
        temp += i * i;
    }
    sink = temp; // Forces the compiler to execute the loop to get the value for sink!

    auto end = std::chrono::high_resolution_clock::now();
    std::chrono::duration<double, std::milli> duration = end - start;
    std::cout << "Duration: " << duration.count() << " ms\n";
}
```

### Method B: Inline Assembly Barriers (GCC/Clang Specific)
Inject inline assembly constraints to trick the compiler into believing that memory was read and written to, preventing it from optimizing out or reordering code.

- **`doNotOptimize(val)`**: Declares that a variable is read by assembly.
- **`clobberMemory()`**: Declares that memory has changed, forcing all registers to reload.

```cpp
template <typename T>
inline void doNotOptimize(T&& val) {
    // Tells the compiler that the variable val is passed to assembly instruction
    asm volatile("" : "+g"(val)); 
}

inline void clobberMemory() {
    // Forces the compiler to assume all memory is modified
    asm volatile("" : : : "memory"); 
}

void benchmarkWithAssembly() {
    auto start = std::chrono::high_resolution_clock::now();

    int sum = 0;
    for (int i = 0; i < 100'000'000; ++i) {
        sum += i * i;
    }
    doNotOptimize(sum); // Prevents sum from being optimized away

    auto end = std::chrono::high_resolution_clock::now();
    std::chrono::duration<double, std::milli> duration = end - start;
}
```

---

*<- [[09_Perf_SIMD_Basics|SIMD Basics]] · [[09_Perf_Memory_Alignment_and_Padding|Memory Alignment & Padding ->]]*
