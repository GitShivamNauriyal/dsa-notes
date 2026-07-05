---
tags: [cpp, stl, algorithms, complexity-bounds, heap-algorithms, binary-search, stable-partition]
links: ["[[05_STL_Index]]", "[[05_STL_Lambdas_Deep_Dive]]", "[[05_STL_Containers_and_Iterator_Invalidation]]"]
---

# STL & Modern Features -- Algorithms Library

*<- [[05_STL_Lambdas_Deep_Dive|Lambdas Deep Dive]] · [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation ->]]*

---

## 1. Numeric & Transformation Algorithms (Complexity: $O(N)$)

### A. `std::accumulate` (Reduction)
Accumulates a range into a single value. It takes a starting value and optionally a custom binary reduction operator.

```cpp
#include <numeric>
#include <vector>

void demoAccumulate() {
    std::vector<int> nums = {1, 2, 3, 4};
    int sum = std::accumulate(nums.begin(), nums.end(), 0); // 10
    
    // Custom multiplication reduction: Product of range
    int product = std::accumulate(nums.begin(), nums.end(), 1, [](int acc, int x) {
        return acc * x;
    }); // 24
}
```

### B. `std::transform` (Mapping)
Applies a function to elements of a source range and writes them to a destination iterator.

```cpp
#include <algorithm>
#include <vector>

void demoTransform() {
    std::vector<int> nums = {1, 2, 3};
    std::vector<int> results;
    
    // std::back_inserter dynamically calls push_back on 'results'
    std::transform(nums.begin(), nums.end(), std::back_inserter(results), [](int x) {
        return x * 10;
    }); // results: {10, 20, 30}
}
```

---

## 2. Partitioning Algorithms

Partitions elements inside a range such that all elements satisfying a boolean predicate are moved to the front.

- **`std::partition`**: Fast, **unstable** (violates original relative order of elements).
  - **Complexity**: $O(N)$ swaps and applications of the predicate.
- **`std::stable_partition`**: **Stable** (preserves original relative order).
  - **Complexity**: $O(N)$ if temporary memory buffer is allocated; falls back to $O(N \log N)$ swaps if allocation fails.

```cpp
#include <algorithm>
#include <vector>

void demoPartition() {
    std::vector<int> nums = {1, 5, 2, 8, 3, 7};
    // Move all evens to the front preserving order
    std::stable_partition(nums.begin(), nums.end(), [](int x) {
        return x % 2 == 0;
    }); // nums: {2, 8, 1, 5, 3, 7}
}
```

---

## 3. Binary Search Helpers (Sorted Ranges Only)

> [!WARNING]
> **Complexity Trap**: If the container has **Random Access Iterators** (like `std::vector` or `std::deque`), binary search operates in $O(\log N)$ steps. 
> If the container has **Forward/Bidirectional Iterators** (like `std::list` or `std::set`), the iterator must be incremented sequentially, resulting in **$O(N)$ runtime steps**, even though comparison counts remain $O(\log N)$.

- **`std::lower_bound(first, last, val)`**: Returns an iterator to the first element $\ge \text{val}$ (insertion point).
- **`std::upper_bound(first, last, val)`**: Returns an iterator to the first element $> \text{val}$.

```cpp
#include <algorithm>
#include <vector>

void demoBinarySearch() {
    std::vector<int> sorted = {10, 20, 20, 30, 40};
    
    auto lb = std::lower_bound(sorted.begin(), sorted.end(), 20); // index 1 (first 20)
    auto ub = std::upper_bound(sorted.begin(), sorted.end(), 20); // index 3 (value 30)
    
    // Number of elements equal to 20:
    auto count = std::distance(lb, ub); // 2
}
```

---

## 4. In-Place Heap Algorithms (Complexity: $O(\log N)$ / $O(N)$)

Heaps are typically managed using `std::priority_queue`. However, you can manage elements of a raw `std::vector` as a heap directly using in-place heap functions.

- **`std::make_heap`**: Rearranges a range into a max-heap in-place.
  - **Complexity**: Linear $O(N)$ comparisons.
- **`std::push_heap`**: Assumes an element was appended to the back of the vector, and "bubbles it up" to restore heap properties.
  - **Complexity**: $O(\log N)$ comparisons.
- **`std::pop_heap`**: Swaps the maximum element at the root with the last element in the range, then bubbles down to restore heap properties. 
  - **Note**: This does not remove the element; it only moves it to the back. You must call `pop_back()` on the vector afterwards!
  - **Complexity**: $O(\log N)$ comparisons.

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

void demoHeap() {
    std::vector<int> v = {3, 1, 4, 1, 5, 9};

    // 1. Convert vector to max-heap in-place
    std::make_heap(v.begin(), v.end()); // v: {9, 5, 4, 1, 1, 3} (valid max-heap layout)

    // 2. Insert new element
    v.push_back(6);
    std::push_heap(v.begin(), v.end()); // restore heap: 6 bubbles up

    // 3. Retrieve and remove max element (9)
    std::pop_heap(v.begin(), v.end()); // moves 9 to the back of vector
    int maxVal = v.back();             // 9
    v.pop_back();                      // actually removes it from vector
}
```

---

*<- [[05_STL_Lambdas_Deep_Dive|Lambdas Deep Dive]] · [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation ->]]*
