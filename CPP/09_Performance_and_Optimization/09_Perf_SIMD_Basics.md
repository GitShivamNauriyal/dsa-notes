---
tags: [cpp, performance, simd, avx, sse, auto-vectorization, intrinsics]
links: ["[[09_Perf_Index]]", "[[09_Perf_Branch_Prediction_and_Branchless_Code]]", "[[09_Perf_Benchmarking_and_Profiling]]"]
---

# Performance & Optimization -- SIMD Basics

*<- [[09_Perf_Branch_Prediction_and_Branchless_Code|Branch Prediction & Branchless Code]] · [[09_Perf_Benchmarking_and_Profiling|Benchmarking & Profiling ->]]*

---

**SIMD (Single Instruction, Multiple Data)** is a hardware architecture that allows a single CPU execution instruction to operate on multiple data points simultaneously. This data-level parallelism is crucial for high-throughput computations (graphics, DSP, AI, and scientific computing).

---

## 1. SIMD Register Families

Modern CPUs contain wide vector registers dedicated to SIMD execution:

- **SSE (Streaming SIMD Extensions)**:
  - Register Width: **128 bits**.
  - Capacity: Can process **4 floats** (32-bit) or **2 doubles** (64-bit) simultaneously.
- **AVX / AVX2 (Advanced Vector Extensions)**:
  - Register Width: **256 bits**.
  - Capacity: Can process **8 floats** or **4 doubles** simultaneously.
- **AVX-512**:
  - Register Width: **512 bits**.
  - Capacity: Can process **16 floats** or **8 doubles** simultaneously.

---

## 2. Auto-Vectorization & Compiler Flags

If you write standard loops, compilers (GCC, Clang) can automatically vectorize them (translating them into SIMD instructions) under these conditions:
1. Optimizations are enabled (e.g. **`-O3`**).
2. Target instruction sets are specified (e.g. **`-mavx2`** or **`-march=native`**).
3. **No Loop-Carried Dependencies**: An iteration must not rely on the result of a previous iteration (e.g. `data[i] = data[i-1] + 1` cannot be vectorized).
4. Contiguous memory access (no pointer aliasing or indirect indexing).

```cpp
// GCC/Clang with -O3 -mavx2 will auto-vectorize this loop into AVX2 parallel instructions
void addArrays(float* __restrict a, float* __restrict b, float* __restrict c, int size) {
    for (int i = 0; i < size; ++i) {
        c[i] = a[i] + b[i];
    }
}
```

---

## 3. Explicit SIMD Programming using Intrinsics

When the compiler fails to auto-vectorize complex code, you can write SIMD operations manually using hardware **intrinsics** (compiler-exposed C functions that map directly to SIMD assembly instructions).

### Intrinsics Headers:
- `<xmmintrin.h>`: SSE
- `<emmintrin.h>`: SSE2
- `<immintrin.h>`: All Intel/AMD SIMD extensions (SSE to AVX-512)

### SSE and AVX Array Addition Comparison
```cpp
#include <immintrin.h>

// 1. SSE Vectorized Addition (4 floats at a time)
void addSSE(const float* a, const float* b, float* c, int size) {
    for (int i = 0; i < size; i += 4) {
        // Load 128-bit chunks (4 floats) from memory
        __m128 va = _mm_loadu_ps(&a[i]);
        __m128 vb = _mm_loadu_ps(&b[i]);
        
        // Perform parallel float addition
        __m128 vc = _mm_add_ps(va, vb);
        
        // Store result back to memory
        _mm_storeu_ps(&c[i], vc);
    }
}

// 2. AVX Vectorized Addition (8 floats at a time)
void addAVX(const float* a, const float* b, float* c, int size) {
    for (int i = 0; i < size; i += 8) {
        // Load 256-bit chunks (8 floats)
        __m256 va = _mm256_loadu_ps(&a[i]);
        __m256 vb = _mm256_loadu_ps(&b[i]);
        
        // Parallel addition
        __m256 vc = _mm256_add_ps(va, vb);
        
        // Store
        _mm256_storeu_ps(&c[i], vc);
    }
}
```

---

*<- [[09_Perf_Branch_Prediction_and_Branchless_Code|Branch Prediction & Branchless Code]] · [[09_Perf_Benchmarking_and_Profiling|Benchmarking & Profiling ->]]*
