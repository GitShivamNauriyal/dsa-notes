---
tags: [cpp, stl, ranges, views, Cpp20, Cpp23]
links: ["[[05_STL_Index]]", "[[05_STL_Optional_Variant_Tuple]]", "[[05_STL_Problems]]"]
---

# STL & Modern Features -- Ranges & Views

*<- [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple]] · [[05_STL_Problems|Problems ->]]*

---

## 1. What are Ranges & Views?

C++20 **Ranges** allow us to pass container objects directly to algorithms instead of writing verbose iterator pairs (like `std::sort(v.begin(), v.end())` $\to$ `std::ranges::sort(v)`).

**Views** are lightweight, non-owning, lazily-evaluated wrappers over ranges:
- **Lazy Evaluation**: Elements are computed on-the-fly only when iterated over. No temporary vectors or memory allocations are made.
- **Pipe Operator (`|`)**: Allows composing multiple transformations cleanly.

```cpp
#include <vector>
#include <ranges>
#include <iostream>

void demoRanges() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6};

    // Filter evens and square them (lazily computed!)
    auto result = nums 
                | std::views::filter([](int x) { return x % 2 == 0; })
                | std::views::transform([](int x) { return x * x; });

    for (int val : result) {
        std::cout << val << " "; // Prints: 4 16 36
    }
}
```

---

## 2. C++23 Ranges & Views Enhancements

C++23 introduced highly requested views to match modern functional language properties:

### A. `std::views::enumerate` (C++23)
Zips a counter index with the elements of a range (identical to Python's `enumerate`).

```cpp
#include <vector>
#include <ranges>
#include <iostream>
#include <string>

void demoEnumerate() {
    std::vector<std::string> names = {"Alice", "Bob", "Charlie"};

    // C++23 views::enumerate
    for (auto [index, name] : names | std::views::enumerate) {
        std::cout << index << ": " << name << "\n";
        // Prints:
        // 0: Alice
        // 1: Bob
        // 2: Charlie
    }
}
```

### B. `std::views::zip` (C++23)
Combines multiple ranges into a single range of tuples.

```cpp
#include <vector>
#include <ranges>
#include <iostream>

void demoZip() {
    std::vector<int> ids = {1, 2, 3};
    std::vector<char> grades = {'A', 'B', 'A'};

    // C++23 views::zip
    for (auto [id, grade] : std::views::zip(ids, grades)) {
        std::cout << "Student " << id << " got " << grade << "\n";
    }
}
```

---

*<- [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple]] · [[05_STL_Problems|Problems ->]]*
