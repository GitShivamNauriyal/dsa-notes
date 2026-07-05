---
tags: [cpp, stl, algorithms, transform, accumulate, lower-bound]
links: ["[[05_STL_Index]]", "[[05_STL_Lambdas_Deep_Dive]]", "[[05_STL_Containers_and_Iterator_Invalidation]]"]
---

# STL & Modern Features -- Algorithms Library

*<- [[05_STL_Lambdas_Deep_Dive|Lambdas Deep Dive]] · [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation ->]]*

---

## 1. Numeric & Transformation Algorithms

The `<algorithm>` and `<numeric>` headers contain highly optimized, bounds-safe functions to manipulate ranges.

### A. `accumulate` (Reduction)
Reduces a range to a single value by applying a binary operator (sums by default).

```cpp
#include <numeric>
#include <vector>

void demoAccumulate() {
    std::vector<int> nums = {1, 2, 3, 4};
    
    // Sum range
    int sum = std::accumulate(nums.begin(), nums.end(), 0); // 10
    
    // Custom operator: Product of range
    int product = std::accumulate(nums.begin(), nums.end(), 1, [](int acc, int x) {
        return acc * x;
    }); // 24
}
```

### B. `transform` (Mapping)
Applies a function to a range and writes the outputs to another destination range.

```cpp
#include <algorithm>
#include <vector>

void demoTransform() {
    std::vector<int> nums = {1, 2, 3};
    std::vector<int> squares(nums.size());
    
    // Calculate squares and write to 'squares' vector
    std::transform(nums.begin(), nums.end(), squares.begin(), [](int x) {
        return x * x;
    });
}
```

---

## 2. Partitioning

Partitioning reorders a range based on a boolean predicate, moving elements that satisfy the predicate to the front.
- **`std::partition`**: Reorders elements. Fast but **unstable** (relative order of elements inside partitions may change).
- **`std::stable_partition`**: Reorders elements while **preserving** relative order. Requires auxiliary memory allocation under the hood.

```cpp
#include <algorithm>
#include <vector>

void demoPartition() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6};
    
    // Partition even numbers to the front
    std::stable_partition(nums.begin(), nums.end(), [](int x) {
        return x % 2 == 0;
    });
    // nums is now: {2, 4, 6, 1, 3, 5} (evens preserved relative order, odds preserved relative order)
}
```

---

## 3. Binary Search Helpers

When searching inside a **sorted range**:
- **`std::lower_bound(first, last, val)`**: Returns an iterator pointing to the first element that is **greater than or equal** to `val` ($\ge \text{val}$).
- **`std::upper_bound(first, last, val)`**: Returns an iterator pointing to the first element that is **strictly greater** than `val` ($> \text{val}$).

```cpp
#include <algorithm>
#include <vector>

void demoBounds() {
    std::vector<int> data = {10, 20, 20, 20, 30, 40};
    
    auto lb = std::lower_bound(data.begin(), data.end(), 20); // Points to index 1 (first 20)
    auto ub = std::upper_bound(data.begin(), data.end(), 20); // Points to index 4 (value 30)
    
    // Number of occurrences of 20:
    auto count = std::distance(lb, ub); // 3
}
```

---

*<- [[05_STL_Lambdas_Deep_Dive|Lambdas Deep Dive]] · [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation ->]]*
