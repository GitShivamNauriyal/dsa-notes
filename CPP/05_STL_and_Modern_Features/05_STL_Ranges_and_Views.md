---
tags: [cpp, stl, ranges, views, Cpp20, Cpp23, ranges-to, dangling-ranges]
links: ["[[05_STL_Index]]", "[[05_STL_Optional_Variant_Tuple]]", "[[05_STL_Problems]]"]
---

# STL & Modern Features -- Ranges & Views

*<- [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple]] · [[05_STL_Problems|Problems ->]]*

---

## 1. What are Ranges & Views?

C++20 **Ranges** allow algorithms to operate directly on containers rather than raw iterator pairs (e.g. `std::ranges::sort(v)` instead of `std::sort(v.begin(), v.end())`).

**Views** are lightweight, non-owning, lazily-evaluated wrappers over ranges:
- **Lazy Evaluation**: Calculations are computed only when the view is iterated. No intermediate copies or allocations occur, making them highly efficient.
- **Pipe Operator (`|`)**: Allows composing multiple transformations.

```cpp
#include <vector>
#include <ranges>
#include <iostream>

void demoRanges() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6};

    // Filter evens and square them (lazily evaluated)
    auto result = nums 
                | std::views::filter([](int x) { return x % 2 == 0; })
                | std::views::transform([](int x) { return x * x; });

    for (int val : result) {
        std::cout << val << " "; // Prints: 4 16 36
    }
}
```

---

## 2. Converting Views to Containers: `std::ranges::to` (C++23)

In C++20, converting a lazy view pipeline back into a concrete container (like `std::vector` or `std::set`) was awkward, requiring you to write manual loops or insert iterators. 
C++23 introduced **`std::ranges::to`** to allow direct materialization.

```cpp
#include <vector>
#include <ranges>
#include <set>

void demoMaterialize() {
    std::vector<int> nums = {5, 2, 8, 2, 9};

    // Filter and materialize directly into a std::set (which sorts and removes duplicates)
    auto uniqueEvens = nums 
                     | std::views::filter([](int x) { return x % 2 == 0; })
                     | std::ranges::to<std::set<int>>(); // C++23: Materialize!
}
```

---

## 3. The Compile-Time Dangling Protection

If you pass a temporary (rvalue) container to a traditional STL algorithm, it compiles, but returning an iterator to it leaves you with a dangling pointer when the temporary dies.
C++20 ranges block this at compile-time by returning a wrapper type called **`std::ranges::dangling`** instead of an iterator if the passed container is an rvalue.

```cpp
#include <algorithm>
#include <vector>
#include <ranges>

std::vector<int> getTemps() { return {1, 2, 3}; }

void demoDangling() {
    // getTemps() is an rvalue temporary
    auto it = std::ranges::find(getTemps(), 2);
    
    // it is of type 'std::ranges::dangling', NOT 'std::vector<int>::iterator'!
    // Attempting to dereference '*it' fails to compile, protecting you from dangling memory bugs!
    // int val = *it; // COMPILER ERROR!
}
```

---

## 4. C++23 View Composition Enhancements

### A. `std::views::enumerate` (C++23)
Pairs each element with its 0-based iteration index.

```cpp
#include <vector>
#include <ranges>
#include <iostream>
#include <string>

void demoEnumerate() {
    std::vector<std::string> names = {"Alice", "Bob"};

    for (auto [index, name] : names | std::views::enumerate) {
        std::cout << index << ": " << name << "\n";
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
    std::vector<int> ids = {1, 2};
    std::vector<char> grades = {'A', 'B'};

    for (auto [id, grade] : std::views::zip(ids, grades)) {
        std::cout << id << " -> " << grade << "\n";
    }
}
```

---

*<- [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple]] · [[05_STL_Problems|Problems ->]]*
