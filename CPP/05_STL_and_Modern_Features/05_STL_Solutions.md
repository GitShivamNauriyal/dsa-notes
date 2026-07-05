---
tags: [cpp, stl, solutions]
links: ["[[05_STL_Index]]", "[[05_STL_Problems]]", "[[05_STL_Tricky]]"]
---

# STL & Modern Features -- Solutions

*<- [[05_STL_Problems|Problems]] · [[05_STL_Tricky|Tricky STL Gotchas ->]]*

---

## Tier 1 -- Concept Checks

### Solution 1.1: Iterator Invalidation
- **Outcome**: **Undefined Behavior (UB)**.
- **Explanation**: 
  - `v` is initialized with size 3 and capacity 3.
  - `push_back(4)` causes the vector to exceed its capacity, triggering a reallocation.
  - The vector allocates a new, larger memory block elsewhere, copies the elements, and deallocates the original block.
  - `it` still points to the old, deallocated memory block. Dereferencing `*it` reads deallocated memory (dangling pointer UB).

---

### Solution 1.2: Variant Retrieval Safety
1. `std::get<int>(v)`: **Throws `std::bad_variant_access` exception** because the active type inside the variant is `std::string`, not `int`.
2. `std::get_if<int>(&v)`: **Returns `nullptr`** (does not throw) because we passed a pointer to the variant, allowing safe conditional checks without exceptions.

---

## Tier 2 -- Implementation

### Solution 2.1: Extract Map Keys using `transform`

```cpp
#include <map>
#include <vector>
#include <algorithm>
#include <iterator>

template <typename K, typename V>
std::vector<K> extractKeys(const std::map<K, V>& m) {
    std::vector<K> keys;
    // Pre-allocate space for performance
    keys.reserve(m.size()); 
    
    // std::transform extracts the first element (key) of each map node pair
    std::transform(m.begin(), m.end(), std::back_inserter(keys), [](const auto& pair) {
        return pair.first;
    });
    return keys;
}
```

---

## Tier 3 -- Interview-Level

### Solution 3.1: Top-K Frequent Elements (STL heap)

```cpp
#include <vector>
#include <unordered_map>
#include <queue>
#include <utility>

class FrequencyTracker {
    std::unordered_map<int, int> counts;
public:
    void add(int num) {
        counts[num]++;
    }

    std::vector<int> getTopK(int k) {
        // Min-heap storing pair: {frequency, number}
        // Configured with custom lambda comparator
        auto cmp = [](const std::pair<int, int>& a, const std::pair<int, int>& b) {
            return a.first > b.first; // min-heap: higher frequencies sink
        };
        std::priority_queue<std::pair<int, int>, std::vector<std::pair<int, int>>, decltype(cmp)> minHeap(cmp);

        for (const auto& [num, freq] : counts) {
            minHeap.push({freq, num});
            if ((int)minHeap.size() > k) {
                minHeap.pop(); // pop element with lowest frequency
            }
        }

        std::vector<int> result;
        while (!minHeap.empty()) {
            result.push_back(minHeap.top().second);
            minHeap.pop();
        }
        return result;
    }
};
```

---

### Solution 3.2: Custom String Comparators for Sorting

```cpp
#include <vector>
#include <string>
#include <algorithm>

void sortStrings(std::vector<std::string>& arr) {
    std::sort(arr.begin(), arr.end(), [](const std::string& a, const std::string& b) {
        if (a.size() != b.size()) {
            return a.size() < b.size(); // sort by length ascending
        }
        return a < b; // sort alphabetically
    });
}
```

---

## Tier 4 -- Systems / Placement-Hard

### Solution 4.1: Custom Step Iterator

```cpp
#include <vector>
#include <iostream>
#include <iterator>

template <typename Iterator>
class StepIterator {
    Iterator current;
    std::size_t stride;
public:
    using iterator_category = std::forward_iterator_tag;
    using value_type = typename std::iterator_traits<Iterator>::value_type;
    using difference_type = typename std::iterator_traits<Iterator>::difference_type;
    using pointer = typename std::iterator_traits<Iterator>::pointer;
    using reference = typename std::iterator_traits<Iterator>::reference;

    StepIterator(Iterator it, std::size_t s) : current(it), stride(s) {}

    // Dereference
    reference operator*() const { return *current; }
    pointer operator->() const { return &(*current); }

    // Increment: move forward by stride S
    StepIterator& operator++() {
        current += stride;
        return *this;
    }

    StepIterator operator++(int) {
        StepIterator temp = *this;
        current += stride;
        return temp;
    }

    // Comparison (stops correctly even if end is bypassed)
    bool operator==(const StepIterator& other) const {
        return current >= other.current; // Greater-than-or-equal check to prevent infinite loop overshoot
    }

    bool operator!=(const StepIterator& other) const {
        return current < other.current;
    }
};

void verifyStepIterator() {
    std::vector<int> v = {10, 20, 30, 40, 50, 60, 70, 80};
    
    // Iterate over v with stride 3 (index 0, 3, 6)
    StepIterator<std::vector<int>::iterator> start(v.begin(), 3);
    StepIterator<std::vector<int>::iterator> end(v.end(), 3);

    for (auto it = start; it != end; ++it) {
        std::cout << *it << " "; // Outputs: 10 40 70
    }
    std::cout << "\n";
}
```

---

*<- [[05_STL_Problems|Problems]] · [[05_STL_Tricky|Tricky STL Gotchas ->]]*
