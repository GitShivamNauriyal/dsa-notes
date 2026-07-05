---
tags: [cpp, performance, branch-prediction, branchless-code, pipeline-flush]
links: ["[[09_Perf_Index]]", "[[09_Perf_Cache_Locality_and_Data_Layout]]", "[[09_Perf_SIMD_Basics]]"]
---

# Performance & Optimization -- Branch Prediction & Branchless Code

*<- [[09_Perf_Cache_Locality_and_Data_Layout|Cache Locality & Data Layout]] · [[09_Perf_SIMD_Basics|SIMD Basics ->]]*

---

Modern CPUs execute instructions concurrently inside multi-stage pipelines. To prevent the pipeline from stalling, the CPU's hardware **Branch Predictor** guesses the outcome of conditional branches (`if-else` blocks).
- **Correct Guess**: Zero-overhead.
- **Misprediction**: The CPU flushes the pipeline and re-fetches the correct instruction path, wasting **15 to 20 clock cycles**.

---

## 1. The Sorted vs Unsorted Array Trap

A classic demonstration of branch prediction is summing elements in an array that are greater than a threshold.
- If the array is **unsorted**, the condition `x > 128` alternates randomly. The branch predictor guesses incorrectly roughly 50% of the time.
- If the array is **sorted**, the condition is `false` for the first half and `true` for the second. The branch predictor quickly recognizes the pattern and makes 100% correct guesses.
- **Result**: The sorted array loop executes up to **3 times faster** than the unsorted array loop, even though they perform the exact same work!

```cpp
#include <vector>
#include <algorithm>

long long sumThreshold(std::vector<int>& data) {
    long long sum = 0;
    for (int x : data) {
        if (x >= 128) { // Branch statement
            sum += x;
        }
    }
    return sum;
}

void demoPrediction(std::vector<int>& data) {
    // 1. Unsorted run
    long long s1 = sumThreshold(data);

    // 2. Sorted run
    std::sort(data.begin(), data.end());
    long long s2 = sumThreshold(data); // Executed 3x faster!
}
```

---

## 2. Branchless Programming Techniques

When processing random/unpredictable data, you can optimize hot loops by replacing branches with **branchless code** that compiles down to assembly instructions like **`CMOV`** (Conditional Move) or uses bitwise arithmetic.

### A. Branchless Value Selection
A typical conditional clamp can be rewritten algebraically:

```cpp
// 1. Branched approach
int clampBranched(int val, int limit) {
    if (val > limit) return limit;
    return val;
}

// 2. Branchless approach using arithmetic
int clampBranchless(int val, int limit) {
    bool cond = val > limit;
    // If cond is 1: limit + 0 * (val - limit) = limit
    // If cond is 0: limit + 1 * (val - limit) = val
    return limit + (!cond) * (val - limit);
}
```

### B. Branchless Element Summation
We can rewrite the threshold sum loop from earlier to be branchless by multiplying the value by the boolean condition:

```cpp
long long sumThresholdBranchless(const std::vector<int>& data) {
    long long sum = 0;
    for (int x : data) {
        // True compiles to 1, False to 0.
        // No branch instructions are generated!
        sum += x * (x >= 128); 
    }
    return sum;
}
```

### C. Bitwise Max (Zero Branching)
Finding the maximum of two signed integers without branching using bitwise shifts (assuming 32-bit integers):

```cpp
int maxBranchless(int a, int b) {
    int diff = a - b;
    // If diff is negative, (diff >> 31) is -1 (all bits 1), otherwise 0
    int mask = diff >> 31;
    // Returns a if a > b, else b
    return a - (diff & mask);
}
```

---

*<- [[09_Perf_Cache_Locality_and_Data_Layout|Cache Locality & Data Layout]] · [[09_Perf_SIMD_Basics|SIMD Basics ->]]*
