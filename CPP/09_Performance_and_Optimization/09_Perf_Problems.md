---
tags: [cpp, performance, problems, exercises]
links: ["[[09_Perf_Index]]", "[[09_Perf_Small_Object_Optimizations]]", "[[09_Perf_Solutions]]"]
---

# Performance & Optimization -- Problems

*<- [[09_Perf_Small_Object_Optimizations|Small Object Optimizations]] · [[09_Perf_Solutions|Solutions ->]]*

---

Identify bottlenecks and implement optimized versions for the following performance challenges.

---

### Problem 1: Cache-Optimized Matrix Multiplication
Suppose we have a matrix multiplication algorithm:

```cpp
#include <vector>

using Matrix = std::vector<std::vector<double>>;

Matrix multiplyBranched(const Matrix& A, const Matrix& B, int N) {
    Matrix C(N, Matrix::value_type::value_type(N, 0.0));
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            for (int k = 0; k < N; ++k) {
                C[i][j] += A[i][k] * B[k][j]; // Column-major access to B!
            }
        }
    }
    return C;
}
```

- **The Problem**: Accessing `B[k][j]` inside the innermost loop jumps through memory, creating a cache miss on almost every lookup.
- **The Challenge**: Optimize this function to achieve high cache locality. (Hint: Transpose matrix `B` first, or reorder the loop indices).

---

### Problem 2: Branchless Conditional Accumulation
Optimize the following branched loop to run completely branchless, ensuring there are no branch misprediction stalls when processing random data.

```cpp
#include <vector>

long long processBranched(const std::vector<int>& data) {
    long long sum = 0;
    for (int x : data) {
        if (x % 2 == 0) {
            sum += x * 3;
        } else {
            sum -= x * 2;
        }
    }
    return sum;
}
```

---

### Problem 3: Explicit SIMD Float Multiplication
You have two float arrays `A` and `B` of size $N$ (where $N$ is guaranteed to be a multiple of 8).
Write a function `multiplyAVX` that:
- Uses AVX intrinsics (from `<immintrin.h>`) to multiply elements of `A` and `B` in groups of 8 floats simultaneously.
- Stores the output in float array `C`.

---

### Problem 4: Structure Layout Size Optimization
Arrange the order of members in the structure below to minimize its memory footprint (reduce padding bytes), and write a static assertion checking the size.

```cpp
struct WastefulStruct {
    char a;
    double b;
    bool c;
    int d;
    char e;
};
```

---

*<- [[09_Perf_Small_Object_Optimizations|Small Object Optimizations]] · [[09_Perf_Solutions|Solutions ->]]*
