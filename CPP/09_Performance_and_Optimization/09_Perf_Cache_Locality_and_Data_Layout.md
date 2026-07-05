---
tags: [cpp, performance, cache-locality, aos-soa, matrix-traversal]
links: ["[[09_Perf_Index]]", "[[09_Perf_Branch_Prediction_and_Branchless_Code]]"]
---

# Performance & Optimization -- Cache Locality & Data Layout

*<- [[09_Perf_Index|Index]] · [[09_Perf_Branch_Prediction_and_Branchless_Code|Branch Prediction & Branchless Code ->]]*

---

Modern CPU speed is heavily bottlenecked by memory latency. Accessing RAM takes $\sim 100\text{ns}$ ($\sim 200\text{--}300$ clock cycles), while accessing L1 cache takes $\sim 1\text{ns}$ ($\sim 3\text{--}4$ clock cycles). 
To hide this latency, CPUs load data in contiguous chunks called **cache lines** (typically 64 bytes). Structuring data layouts for cache friendliness is critical.

---

## 1. AoS vs SoA Layouts

When organizing a collection of multi-member objects, you can use either an **Array of Structures (AoS)** or a **Structure of Arrays (SoA)**.

### AoS (Array of Structures)
- **Layout**: `[ {x, y, z, color}, {x, y, z, color}, ... ]`
- **Use Case**: Excellent for object-oriented systems where you process all fields of a single object at the same time.
- **Problem**: If you only want to update the `x` coordinates of all elements, the CPU must still load the unused `y`, `z`, and `color` fields into the cache line, wasting cache space and memory bandwidth.

### SoA (Structure of Arrays)
- **Layout**: `{ [x, x, ...], [y, y, ...], [z, z, ...], [color, color, ...] }`
- **Use Case**: Excellent for math-heavy processing, SIMD vectorization, and cache locality.
- **Advantage**: Iterating over `x` loads only contiguous `x` coordinates into the cache line. No bytes are wasted.

### AoS vs SoA Memory Layout (ASCII Diagram)
```
  AoS (Unused values loaded into Cache Line):
  ┌──────────────────────────────────────────────────────────┐
  │ x0 │ y0 │ z0 │ color0 │ x1 │ y1 │ z1 │ color1 │ x2 ...   │
  └──────────────────────────────────────────────────────────┘
   ◄─────────────────────── Cache Line (64 Bytes) ──────────►
   
  SoA (100% Useful data loaded into Cache Line):
  ┌──────────────────────────────────────────────────────────┐
  │ x0 │ x1 │ x2 │ x3 │ x4 │ x5 │ x6 │ x7 │ x8 │ x9 │ x10 ...  │
  └──────────────────────────────────────────────────────────┘
   ◄─────────────────────── Cache Line (64 Bytes) ──────────►
```

```cpp
#include <vector>

// AoS Layout
struct ParticleAoS {
    float x, y, z;
    int color;
};
std::vector<ParticleAoS> aosParticles;

// SoA Layout
struct ParticleSoA {
    std::vector<float> x;
    std::vector<float> y;
    std::vector<float> z;
    std::vector<int> color;
};
ParticleSoA soaParticles;
```

---

## 2. Matrix Traversal (Row-Major vs Column-Major)

In C and C++, multi-dimensional arrays are stored in **Row-Major Order** (rows are placed contiguously in memory).

- **Row-Major Traversal (Fast)**: Accessing elements as `matrix[i][j]` (outer loop `i`, inner loop `j`) accesses memory sequentially. Cache lines are utilized optimally.
- **Column-Major Traversal (Slow)**: Accessing elements as `matrix[j][i]` (outer loop `i`, inner loop `j`) jumps through memory by offset distances of row sizes, resulting in a cache miss on almost every element!

```cpp
#include <vector>

const int SIZE = 2048;
std::vector<std::vector<int>> matrix(SIZE, std::vector<int>(SIZE, 1));

// Row-Major Traversal: Cache-Friendly!
long long traverseRowMajor() {
    long long sum = 0;
    for (int i = 0; i < SIZE; ++i) {
        for (int j = 0; j < SIZE; ++j) {
            sum += matrix[i][j]; // Contiguous access
        }
    }
    return sum;
}

// Column-Major Traversal: Cache-Destroyer!
long long traverseColumnMajor() {
    long long sum = 0;
    for (int j = 0; j < SIZE; ++j) {
        for (int i = 0; i < SIZE; ++i) {
            sum += matrix[i][j]; // Strided access (jumping SIZE elements)
        }
    }
    return sum;
}
```

---

*<- [[09_Perf_Index|Index]] · [[09_Perf_Branch_Prediction_and_Branchless_Code|Branch Prediction & Branchless Code ->]]*
