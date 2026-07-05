---
tags: [cpp, performance, alignment, padding, struct-packing, alignas]
links: ["[[09_Perf_Index]]", "[[09_Perf_Benchmarking_and_Profiling]]", "[[09_Perf_Move_Semantics_for_Performance]]"]
---

# Performance & Optimization -- Memory Alignment & Padding

*<- [[09_Perf_Benchmarking_and_Profiling|Benchmarking & Profiling]] · [[09_Perf_Move_Semantics_for_Performance|Move Semantics for Performance ->]]*

---

CPUs read memory from RAM in chunks called **words** (typically 4 or 8 bytes). If a variable spans across two word boundaries (**Unaligned Access**), the CPU must perform multiple memory accesses and merge the bytes, degrading performance. 
To prevent this, compilers insert empty bytes called **padding** to align members to their natural boundaries.

---

## 1. Struct Packing Rules & Padding

Every member variable is aligned to a multiple of its size:
- `char`: 1-byte alignment.
- `short`: 2-byte alignment.
- `int`/`float`: 4-byte alignment.
- `double`/pointers: 8-byte alignment.

### Bad Layout vs Good Layout Memory Comparison

```cpp
// 1. Bad layout (wasteful padding)
struct BadLayout {
    char a;      // 1 byte
                 // 7 bytes padding inserted here to align 'b' to 8-byte boundary
    double b;    // 8 bytes
    int c;       // 4 bytes
                 // 4 bytes padding inserted at the end to align the entire struct to 8-byte boundary
}; // sizeof(BadLayout) = 24 bytes!

// 2. Optimized layout (members sorted by size in descending order)
struct GoodLayout {
    double b;    // 8 bytes
    int c;       // 4 bytes
    char a;      // 1 byte
                 // 3 bytes padding inserted at the end to align to 8-byte boundary
}; // sizeof(GoodLayout) = 16 bytes! (Saved 8 bytes per struct!)
```

### ASCII Struct Layout Memory Diagrams
```
  BadLayout (24 Bytes total):
  ┌──────┬──────────────────────┬──────────────────────┬──────────┬──────────┐
  │ a(1) │ Padding (7)          │ b(8)                 │ c(4)     │ Pad(4)   │
  └──────┴──────────────────────┴──────────────────────┴──────────┴──────────┘
  
  GoodLayout (16 Bytes total):
  ┌──────────────────────┬──────────┬──────┬──────────┐
  │ b(8)                 │ c(4)     │ a(1) │ Pad(3)   │
  └──────────────────────┴──────────┴──────┴──────────┘
```

---

## 2. `alignof` and `alignas`

### `alignof`
Retrieves the alignment requirement of any type (e.g. `alignof(double)` is 8).

### `alignas`
Overrides the default alignment to place data on specific hardware boundaries (like cache lines or SIMD registers).

```cpp
#include <iostream>
#include <new>

// Force alignment to 32 bytes (required for AVX register loads)
struct alignas(32) AVXVector {
    float elements[8];
};

void demoAlignment() {
    std::cout << alignof(AVXVector) << "\n"; // Outputs 32
    
    // Force a variable onto a cache line boundary
    alignas(std::hardware_destructive_interference_size) int cacheLineVar = 42;
}
```

---

*<- [[09_Perf_Benchmarking_and_Profiling|Benchmarking & Profiling]] · [[09_Perf_Move_Semantics_for_Performance|Move Semantics for Performance ->]]*
