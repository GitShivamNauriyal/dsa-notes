---
tags: [cpp, stl, containers, iterator-invalidation, vector, deque, unordered-map, rehashing]
links: ["[[05_STL_Index]]", "[[05_STL_Algorithms_Library]]", "[[05_STL_Optional_Variant_Tuple]]"]
---

# STL & Modern Features -- Containers & Iterator Invalidation

*<- [[05_STL_Algorithms_Library|Algorithms Library]] · [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple ->]]*

---

## 1. Sequence Containers: Internal Mechanics

Sequence containers maintain the order of elements you insert.

### A. `std::vector` (Contiguous Allocator)
- **Structure**: Stores elements in a single contiguous heap block.
- **Growth Factor**: When capacity is exceeded, the vector allocates a larger block elsewhere, moves elements, and frees the old block.
  - **GCC/Clang**: Growth factor is **2**.
  - **MSVC**: Growth factor is **1.5** (chosen because a 1.5 multiplier allows the memory allocator to reuse deallocated memory blocks from previous allocations over time, whereas a factor of 2 prevents reuse).

### B. `std::deque` (Double-Ended Queue)
- **Structure**: Lays out segmented blocks of memory connected by a central pointer map.
- **Mechanics**: Does **not** guarantee contiguous memory. However, it supports $O(1)$ indexing (offset calculations via segment blocks) and $O(1)$ inserts/removes at **both** ends without needing to shift existing elements.

```
   Deque Map Array (central block pointers):
   ┌───┬───┬───┬───┬───┐
   │   │   │ * │ * │   │
   └───┼───┼─┬─┼─┬─┴───┘
             │   │
             ▼   ▼
       ┌───┬───┐ ┌───┬───┐
       │ 1 │ 2 │ │ 3 │ 4 │  (Segmented element chunks)
       └───┴───┘ └───┴───┘
```

---

## 2. Associative Containers: Internal Mechanics

### A. `std::map` / `std::set` (Red-Black Trees)
- **Structure**: Balanced Binary Search Trees (specifically, Red-Black Trees).
- **Complexity**: $O(\log N)$ for search, insert, and delete.
- **Iterators**: Stable. Because elements reside in isolated, heap-allocated nodes, inserting items does not relocate other nodes in memory. Pointers to node elements remain valid indefinitely unless specifically deleted.

### B. `std::unordered_map` / `std::unordered_set` (Hash Tables)
- **Structure**: An array of buckets. Each bucket contains a linked list of nodes (collision resolution by chaining).
- **Load Factor**: $\text{load\_factor} = \frac{\text{size()}}{\text{bucket\_count()}}$.
- **Rehashing**: When the load factor exceeds `max_load_factor()`, the container allocates a larger bucket array and re-hashes all elements to new bucket offsets.
  - **Cost**: A heavy $O(N)$ operation that invalidates **all** active iterators.

---

## 3. Iterator Invalidation Matrix

| Container | Insertion Invalidation Rules | Erasure Invalidation Rules |
|---|---|---|
| **`std::vector`** | If capacity changes: **all** iterators/references.<br>Otherwise: only elements at and after insertion point. | Only elements at and after the erased point. |
| **`std::deque`** | Always invalidates **all iterators**.<br>References are preserved if inserting at boundaries. | Always invalidates **all iterators**.<br>References preserved if erasing at boundaries. |
| **`std::list`** | **None** (node pointers remain stable). | Only the iterator of the **erased node** is invalidated. |
| **`std::map` / `set`**| **None** (node pointers remain stable). | Only the iterator of the **erased node** is invalidated. |
| **`std::unordered_map`**| If rehashing occurs: **all iterators** (references stay valid).<br>Otherwise: none. | Only the iterator of the **erased node** is invalidated. |

---

## 4. The Loop-Erasure Bug & Fixes

Modifying a container while iterating through it is a classic systems-programming trap.

### The Bug (Undefined Behavior)
```cpp
#include <vector>

void buggyErase() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    for (auto it = nums.begin(); it != nums.end(); ++it) {
        if (*it % 2 == 0) {
            nums.erase(it); // UB: 'it' is invalidated! The loop increment '++it' now reads garbage.
        }
    }
}
```

### The Correct Fixes:

#### Fix A: Capture the returned iterator
`erase()` returns a new valid iterator pointing to the element immediately following the erased one.

```cpp
#include <vector>

void correctErase() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    auto it = nums.begin();
    while (it != nums.end()) {
        if (*it % 2 == 0) {
            it = nums.erase(it); // Update it with the returned valid iterator (do not increment)
        } else {
            ++it; // Only increment if we didn't erase
        }
    }
}
```

#### Fix B: Uniform Container Erasure (C++20)
C++20 introduced global helper functions `std::erase` and `std::erase_if` which handle loop-erasure logic safely and optimally for all containers.

```cpp
#include <vector>

void modernErase() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    std::erase_if(nums, [](int x) {
        return x % 2 == 0;
    }); // nums is now: {1, 3, 5}
}
```

---

*<- [[05_STL_Algorithms_Library|Algorithms Library]] · [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple ->]]*
