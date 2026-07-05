---
tags: [cpp, performance, solutions]
links: ["[[09_Perf_Index]]", "[[09_Perf_Problems]]", "[[09_Perf_Tricky]]"]
---

# Performance & Optimization -- Solutions

*<- [[09_Perf_Problems|Problems]] · [[09_Perf_Tricky|Tricky Performance Gotchas ->]]*

---

### Solution 1: Cache-Optimized Matrix Multiplication

We can optimize the matrix multiplication algorithm by **transposing matrix B** before executing the multiplications.
- Transposing `B` swaps its rows and columns.
- Accessing `B[k][j]` now becomes `B_T[j][k]`, which accesses memory sequentially (row-major). This improves cache locality.

```cpp
#include <vector>

using Matrix = std::vector<std::vector<double>>;

Matrix multiplyOptimized(const Matrix& A, const Matrix& B, int N) {
    Matrix C(N, Matrix::value_type::value_type(N, 0.0));
    
    // 1. Create a transposed copy of B
    Matrix B_T(N, Matrix::value_type::value_type(N, 0.0));
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            B_T[i][j] = B[j][i];
        }
    }

    // 2. Perform multiplication using B_T
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            double sum = 0.0;
            for (int k = 0; k < N; ++k) {
                // Both A[i][k] and B_T[j][k] access memory sequentially (Row-Major)!
                sum += A[i][k] * B_T[j][k]; 
            }
            C[i][j] = sum;
        }
    }
    return C;
}
```

---

### Solution 2: Branchless Conditional Accumulation

We can write this loop completely branchless using algebraic operations.
- Calculate condition flags: `bool even = (x % 2 == 0)`.
- Use the flags as multipliers: `(even * 3) + (!even * -2)`.

```cpp
#include <vector>

long long processBranchless(const std::vector<int>& data) {
    long long sum = 0;
    for (int x : data) {
        // Compiles down to conditional move (CMOV) or simple arithmetic; zero branching!
        bool even = (x % 2 == 0);
        int multiplier = even * 3 + (!even) * -2;
        sum += static_cast<long long>(x) * multiplier;
    }
    return sum;
}
```

---

### Solution 3: Explicit SIMD Float Multiplication

Using AVX intrinsics `_mm256_loadu_ps`, `_mm256_mul_ps`, and `_mm256_storeu_ps`.

```cpp
#include <immintrin.h>

void multiplyAVX(const float* a, const float* b, float* c, int size) {
    // Process in steps of 8 floats (256 bits)
    for (int i = 0; i < size; i += 8) {
        __m256 va = _mm256_loadu_ps(&a[i]);
        __m256 vb = _mm256_loadu_ps(&b[i]);
        
        __m256 vc = _mm256_mul_ps(va, vb); // Element-wise multiplication
        
        _mm256_storeu_ps(&c[i], vc);
    }
}
```

---

### Solution 4: Structure Layout Size Optimization

Sort structure members by size in descending order to minimize padding:
1. `double` (8 bytes)
2. `int` (4 bytes)
3. `char`, `bool` (1 byte each)

```cpp
#include <cstddef>

struct OptimizedStruct {
    double b; // 8 bytes
    int d;    // 4 bytes
    char a;   // 1 byte
    char e;   // 1 byte
    bool c;   // 1 byte
              // 1 byte padding inserted here to align structure size to 8-byte boundary
}; 

// Verify optimization at compile time
static_assert(sizeof(OptimizedStruct) == 16, "OptimizedStruct layout is not optimal!");
```

---

*<- [[09_Perf_Problems|Problems]] · [[09_Perf_Tricky|Tricky Performance Gotchas ->]]*
