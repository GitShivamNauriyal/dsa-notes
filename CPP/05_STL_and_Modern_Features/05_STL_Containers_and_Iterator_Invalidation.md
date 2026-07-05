---
tags: [cpp, stl, containers, iterator-invalidation, vector, list]
links: ["[[05_STL_Index]]", "[[05_STL_Algorithms_Library]]", "[[05_STL_Optional_Variant_Tuple]]"]
---

# STL & Modern Features -- Containers & Iterator Invalidation

*<- [[05_STL_Algorithms_Library|Algorithms Library]] · [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple ->]]*

---

## 1. Iterator Invalidation

**Iterator Invalidation** occurs when operations that modify a container (like inserting or erasing elements) render existing iterators, pointers, or references to elements invalid. Accessing an invalidated iterator triggers **Undefined Behavior**.

---

## 2. Invalidation Rules Matrix

| Container | Insertion Invalidation Rules | Erasure Invalidation Rules |
|---|---|---|
| **`std::vector`** | If capacity changes: **all** iterators/references.<br>Otherwise: only elements at and after insertion point. | Only elements at and after the erased point. |
| **`std::deque`** | Always invalidates **all iterators**.<br>References are preserved if inserting at boundaries. | Always invalidates **all iterators**.<br>References preserved if erasing at boundaries. |
| **`std::list`** | **None** (other nodes are untouched). | Only the iterator of the **erased node** is invalidated. |
| **`std::map` / `set`**| **None** (node pointers remain stable). | Only the iterator of the **erased node** is invalidated. |
| **`std::unordered_map`**| If rehashing occurs: **all iterators** (references stay valid).<br>Otherwise: none. | Only the iterator of the **erased node** is invalidated. |

---

## 3. The Classic Loop-Erasure Bug & Fix

A common bug in C++ interviews is attempting to erase elements while iterating through a sequence container.

### The Buggy Code:
```cpp
#include <vector>

void buggyErase() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    for (auto it = nums.begin(); it != nums.end(); ++it) {
        if (*it % 2 == 0) {
            nums.erase(it); // UB: 'it' is invalidated! The loop increment '++it' now reads stack garbage.
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

#### Fix B: Erase-Remove Idiom (Modern C++ / C++20 `std::erase`)
- **Pre-C++20**: `nums.erase(std::remove_if(nums.begin(), nums.end(), pred), nums.end());`
- **C++20 (Uniform Container Erasure)**: Use `std::erase` or `std::erase_if` directly. It handles the iterator updates safely under the hood.

```cpp
#include <vector>

void modernErase() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    // C++20: Safe, clean, and optimized
    std::erase_if(nums, [](int x) {
        return x % 2 == 0;
    });
}
```

---

*<- [[05_STL_Algorithms_Library|Algorithms Library]] · [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple ->]]*
