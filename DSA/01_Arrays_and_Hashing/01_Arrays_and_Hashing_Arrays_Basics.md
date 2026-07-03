---
tags: [dsa, arrays, cpp, stl, vector]
links: ["[[01_Arrays_and_Hashing_Index]]", "[[01_Arrays_and_Hashing_Prefix_Sums]]"]
---

# Arrays — Basics

*← [[01_Arrays_and_Hashing_Index|Index]] · [[01_Arrays_and_Hashing_Prefix_Sums|Prefix Sums →]]*

---

## What is an Array?

An array is a **contiguous block of memory** holding elements of the same type. Because elements sit next to each other in memory, accessing any element by index is O(1) — the CPU just computes `base_address + index * sizeof(element)`.

This is the key insight behind almost every array trick: **we can jump to any position instantly**.

```
Index:    0    1    2    3    4
          ↓    ↓    ↓    ↓    ↓
Memory: [ 10 | 20 | 30 | 40 | 50 ]
         ^
         base_address
```

---

## 1. C-Style Arrays vs std::vector

```cpp
#include <vector>
#include <array>
#include <iostream>

int main() {
    // --- C-style array: fixed size, lives on stack ---
    int arr[5] = {1, 2, 3, 4, 5};
    // arr[5] is undefined behavior — no bounds checking!

    // --- array (C++11): fixed size, stack, safe ---
    array<int, 5> a = {1, 2, 3, 4, 5};
    // a.at(5) throws out_of_range — safe version

    // --- vector: dynamic size, heap, preferred in interviews ---
    vector<int> v = {1, 2, 3, 4, 5};
    v.push_back(6);           // O(1) amortized
    v.pop_back();             // O(1)
    int x = v[2];             // O(1), no bounds check
    int y = v.at(2);          // O(1), throws if out of range

    // Size vs Capacity
    // size()     = number of elements currently stored
    // capacity() = memory currently allocated (>= size)
    // When size == capacity and you push_back, vector doubles capacity
    v.reserve(100);           // pre-allocate to avoid repeated reallocation
    v.resize(10, 0);          // resize to 10 elements, fill new ones with 0

    return 0;
}
```

---

## 2. Declaration Patterns (Interview Common Forms)

```cpp
#include <vector>

// 1D vector of size n, initialized to 0
vector<int> v(n, 0);

// 2D vector of size rows x cols, initialized to 0
vector<vector<int>> grid(rows, vector<int>(cols, 0));

// 2D array from input
int rows, cols;
// ... read rows, cols ...
vector<vector<int>> mat(rows, vector<int>(cols));
for (int i = 0; i < rows; i++)
    for (int j = 0; j < cols; j++)
        cin >> mat[i][j];

// Vector of pairs
vector<pair<int,int>> pairs;
pairs.push_back({3, 5});
pairs.emplace_back(3, 5);   // slightly more efficient than push_back

// Vector of tuples
#include <tuple>
vector<tuple<int,int,int>> triples;
triples.emplace_back(1, 2, 3);
auto [a, b, c] = triples[0];  // C++17 structured binding — unpacks tuple
```

---

## 3. Traversal Patterns

```cpp
#include <vector>
#include <iostream>

void traversals(vector<int>& v) {
    int n = v.size();

    // --- Pattern 1: Index-based (most common, gives you index) ---
    for (int i = 0; i < n; i++) {
        cout << v[i];
    }

    // --- Pattern 2: Range-based for (C++11, clean, no index) ---
    for (int x : v) {
        cout << x;
    }

    // --- Pattern 3: Range-based with reference (use when modifying) ---
    for (int& x : v) {
        x *= 2;  // modifies the actual element
    }

    // --- Pattern 4: Reverse traversal ---
    for (int i = n - 1; i >= 0; i--) {
        cout << v[i];
    }

    // --- Pattern 5: Two indices moving toward each other ---
    int left = 0, right = n - 1;
    while (left < right) {
        swap(v[left], v[right]);
        left++;
        right--;
    }
    // ^ This reverses the array in-place in O(n) time, O(1) space
}
```

---

## 4. In-Place Tricks

### Trick 1: Reverse an Array

```cpp
// Why: swap elements symmetrically around the center
// No extra array needed — O(1) space
void reverse(vector<int>& v) {
    int left = 0, right = (int)v.size() - 1;
    while (left < right) {
        swap(v[left], v[right]);
        left++;
        right--;
    }
}
// STL equivalent: reverse(v.begin(), v.end());
```

### Trick 2: Remove Duplicates from Sorted Array (In-Place)

```cpp
// Why: since array is sorted, duplicates are adjacent.
// Use a write pointer that only advances when we see a new value.
int removeDuplicates(vector<int>& v) {
    if (v.empty()) return 0;
    int write = 1;                    // next position to write to
    for (int read = 1; read < (int)v.size(); read++) {
        if (v[read] != v[read - 1]) { // new unique value found
            v[write] = v[read];
            write++;
        }
    }
    return write;                     // length of deduplicated array
}
```

### Trick 3: Rotate Array by k Steps

```cpp
// Why: three reverses = one rotation. No extra space.
// Rotate [1,2,3,4,5] by k=2 → [4,5,1,2,3]
// Step 1: reverse whole array   [5,4,3,2,1]
// Step 2: reverse first k       [4,5,3,2,1]
// Step 3: reverse last n-k      [4,5,1,2,3]  ✓
void rotate(vector<int>& v, int k) {
    int n = v.size();
    k %= n;  // handle k > n
    reverse(v.begin(), v.end());
    reverse(v.begin(), v.begin() + k);
    reverse(v.begin() + k, v.end());
}
```

### Trick 4: Move All Zeros to End

```cpp
// Why: read pointer scans, write pointer only moves when non-zero found
void moveZeros(vector<int>& v) {
    int write = 0;
    for (int read = 0; read < (int)v.size(); read++) {
        if (v[read] != 0) {
            v[write++] = v[read];
        }
    }
    while (write < (int)v.size()) {
        v[write++] = 0;
    }
}
```

---

## 5. C++17 and C++20 Features for Arrays

*First, the standard way. Then the modern shorthand.*

### Structured Bindings (C++17)

```cpp
// Standard way (pre-C++17)
pair<int,int> p = {3, 5};
int first = p.first;
int second = p.second;

// C++17: structured binding — cleaner, same performance
auto [a, b] = p;

// Works with arrays too
int arr[3] = {1, 2, 3};
auto [x, y, z] = arr;

// Works in range-based for with map
map<string, int> freq;
for (auto& [key, val] : freq) {
    // use key and val directly
}
```

### std::span (C++20) — Non-owning Array View

```cpp
#include <span>

// Why: pass a subarray or any contiguous range without copying
// span is a non-owning view — no allocation, just a pointer + size

void process(span<int> s) {  // works with vector, array, raw array
    for (int x : s) cout << x << " ";
}

int main() {
    vector<int> v = {1,2,3,4,5};
    process(v);                        // whole vector
    process(span(v).subspan(1,3)); // elements [1..3], i.e., {2,3,4}
    // ^ subspan(offset, count) — no copy made
}
// Note: span does NOT own the data. The original must stay alive.
```

### Ranges (C++20) — Cleaner Algorithm Calls

```cpp
#include <algorithm>
#include <ranges>
#include <vector>

int main() {
    vector<int> v = {5, 3, 1, 4, 2};

    // Standard way (C++17)
    sort(v.begin(), v.end());

    // C++20 ranges — pass container directly, no .begin()/.end()
    ranges::sort(v);

    // Filter and transform (lazy — no intermediate container)
    // C++20 views: chain operations like a pipeline
    auto even_doubled = v
        | views::filter([](int x){ return x % 2 == 0; })
        | views::transform([](int x){ return x * 2; });
    // ^ Nothing is computed yet — only when you iterate
    for (int x : even_doubled) cout << x << " ";
}
```

### std::erase and std::erase_if (C++20)

```cpp
#include <vector>
#include <algorithm>  // for erase (C++20)

vector<int> v = {1, 2, 3, 2, 4, 2};

// Standard way (erase-remove idiom, C++11/17)
v.erase(remove(v.begin(), v.end(), 2), v.end());

// C++20: one function call
erase(v, 2);             // remove all elements equal to 2
erase_if(v, [](int x){ return x % 2 == 0; }); // remove evens
```

---

## 6. Key STL Functions for Arrays (Interview Must-Know)

```cpp
#include <algorithm>
#include <numeric>
#include <vector>

vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6};

// --- Searching ---
auto it = find(v.begin(), v.end(), 5);      // O(n)
bool has5 = (it != v.end());

// On SORTED arrays:
sort(v.begin(), v.end());
auto lb = lower_bound(v.begin(), v.end(), 4); // first >= 4, O(log n)
auto ub = upper_bound(v.begin(), v.end(), 4); // first > 4,  O(log n)
int count4 = ub - lb;                              // count of 4s

// --- Min/Max ---
int mn = *min_element(v.begin(), v.end());    // O(n)
int mx = *max_element(v.begin(), v.end());    // O(n)
auto [mn2, mx2] = minmax_element(v.begin(), v.end()); // one pass

// --- Sum / Accumulate ---
#include <numeric>
long long total = accumulate(v.begin(), v.end(), 0LL); // 0LL for long long sum

// --- Counting ---
int cnt = count(v.begin(), v.end(), 1);       // count occurrences of 1

// --- Sorting with custom comparator ---
sort(v.begin(), v.end(), greater<int>()); // descending
sort(v.begin(), v.end(), [](int a, int b){ return a > b; }); // same

// --- Permutations ---
sort(v.begin(), v.end()); // must sort first
do {
    // process current permutation
} while (next_permutation(v.begin(), v.end())); // O(n * n!)
```

---

*← [[01_Arrays_and_Hashing_Index|Index]] · [[01_Arrays_and_Hashing_Prefix_Sums|Next: Prefix Sums →]]*
