---
tags: [cpp, stl, problems, exercises]
links: ["[[05_STL_Index]]", "[[05_STL_Ranges_and_Views]]", "[[05_STL_Solutions]]"]
---

# STL & Modern Features -- Problems & Exercises

*<- [[05_STL_Ranges_and_Views|Ranges & Views]] · [[05_STL_Solutions|Solutions ->]]*

---

## Tier 1 -- Concept Checks

### Question 1.1: Iterator Invalidation
Predict if the following code block exhibits Undefined Behavior (UB), compilation error, or safe execution. Explain why:
```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    auto it = v.begin();
    v.push_back(4);
    std::cout << *it << "\n";
    return 0;
}
```

---

### Question 1.2: Variant Retrieval Safety
Given `std::variant<int, std::string> v = "Hello";`, what happens if we execute:
1. `std::get<int>(v)`?
2. `std::get_if<int>(&v)`?

---

## Tier 2 -- Implementation

### Question 2.1: Extract Map Keys using `transform`
Write a function `template <typename K, typename V> std::vector<K> extractKeys(const std::map<K, V>& m)` that extracts all keys from a map and returns them in a vector using `std::transform` and back-inserters.

---

## Tier 3 -- Interview-Level

### Question 3.1: Top-K Frequent Elements (STL heap)
Design and implement a class `FrequencyTracker` that:
- Accepts integers via `void add(int num)`.
- Returns the top $K$ most frequent elements in $O(N \log K)$ time using `std::unordered_map` and a min-heap configured using `std::priority_queue` with a custom lambda comparator.

---

### Question 3.2: Custom String Comparators for Sorting
Write a program that sorts a list of strings such that:
- Strings are sorted by length in ascending order.
- If two strings have equal length, they are sorted alphabetically.
Implement this using `std::sort` with a custom lambda comparator.

---

## Tier 4 -- Systems / Placement-Hard

### Question 4.1: Custom Step Iterator
In scientific data processing, we often need to iterate over memory buffers in steps (strides) rather than sequentially (e.g. read every 3rd pixel).
Design and implement a custom iterator wrapper `StepIterator<Iterator>` that:
- Wraps any standard contiguous iterator (like `std::vector<int>::iterator`).
- Takes a stride parameter $S$ in its constructor.
- Overloads operator `++` to advance the iterator by $S$ elements instead of 1.
- Supports comparison operators (`==`, `!=`) and dereferencing (`*`).
Show how to use it to iterate over a `std::vector<int>`.

---

*<- [[05_STL_Ranges_and_Views|Ranges & Views]] · [[05_STL_Solutions|Solutions ->]]*
