---
tags: [dsa, complexity, big-o, master-theorem]
links: ["[[00_Home]]"]
---

# Complexity Cheatsheet

## 1. Big-O — What It Means

Big-O describes the **worst-case growth rate** of an algorithm as input `n` grows.

| Notation | Name | Example |
|----------|------|---------|
| O(1) | Constant | Array index access, hash map lookup |
| O(log n) | Logarithmic | Binary search, balanced BST ops |
| O(n) | Linear | Single loop over array |
| O(n log n) | Linearithmic | Merge sort, heap sort, `std::sort` |
| O(n²) | Quadratic | Bubble sort, nested loops |
| O(n³) | Cubic | Floyd-Warshall, naive matrix multiply |
| O(2ⁿ) | Exponential | All subsets, brute force recursion |
| O(n!) | Factorial | All permutations, TSP brute force |

### Growth Comparison (n = 1000)

```
O(1)       →  1
O(log n)   →  10
O(n)       →  1,000
O(n log n) →  10,000
O(n²)      →  1,000,000
O(2ⁿ)      →  10^301   (impossible for n>40 in competitive programming)
```

> **Rule of thumb for interviews**: If n ≤ 10^8, O(n) is fine. If n ≤ 10^5, O(n log n) is fine. If n ≤ 500, O(n²) may pass. If n ≤ 20, O(2ⁿ) is acceptable.

---

## 2. Space Complexity

Space complexity counts **extra memory** used (not counting input).

| Case | Example |
|------|---------|
| O(1) extra | In-place two pointer, in-place reversal |
| O(n) extra | Prefix sum array, hash map of n elements |
| O(n²) extra | 2D DP table |
| O(h) = O(log n) | Recursive DFS on balanced tree (stack frames) |
| O(h) = O(n) | Recursive DFS on skewed tree |

---

## 3. Recurrence Relations & Master Theorem

Many divide-and-conquer algorithms produce a recurrence like:

```
T(n) = a * T(n/b) + f(n)
```

Where:
- `a` = number of subproblems at each level
- `b` = factor by which problem size reduces
- `f(n)` = work done outside recursive calls

### Master Theorem — 3 Cases

Compute: `c_crit = log_b(a)`  (i.e., log base b of a)

**Case 1**: `f(n) = O(n^c)` where `c < c_crit`
- Recursion dominates → **T(n) = Θ(n^c_crit)**

**Case 2**: `f(n) = Θ(n^c_crit * log^k(n))` for some k ≥ 0
- Equal work at each level → **T(n) = Θ(n^c_crit * log^(k+1)(n))**
- Most common: k=0 → T(n) = Θ(n^c_crit * log n)

**Case 3**: `f(n) = Ω(n^c)` where `c > c_crit`, and regularity holds
- Outside work dominates → **T(n) = Θ(f(n))**

### Master Theorem — Worked Examples

#### Example A: Binary Search
```
T(n) = 1 * T(n/2) + O(1)
a=1, b=2, f(n)=O(1)=O(n^0)
c_crit = log_2(1) = 0
f(n) = O(n^0) → Case 2 (k=0)
T(n) = Θ(log n)   ✓
```

#### Example B: Merge Sort
```
T(n) = 2 * T(n/2) + O(n)
a=2, b=2, f(n)=O(n)=O(n^1)
c_crit = log_2(2) = 1
f(n) = O(n^1) → Case 2 (k=0)
T(n) = Θ(n log n)   ✓
```

#### Example C: Strassen Matrix Multiplication
```
T(n) = 7 * T(n/2) + O(n²)
a=7, b=2, f(n)=O(n²)
c_crit = log_2(7) ≈ 2.807
f(n) = O(n^2) where 2 < 2.807 → Case 1
T(n) = Θ(n^2.807)   ✓  (faster than naive O(n³))
```

#### Example D: Naive Matrix Multiply (3 nested loops)
```
T(n) = 8 * T(n/2) + O(n²)
a=8, b=2, f(n)=O(n²)
c_crit = log_2(8) = 3
f(n) = O(n^2) where 2 < 3 → Case 1
T(n) = Θ(n³)   ✓
```

#### Example E: Quick Select (average case)
```
T(n) = T(n/2) + O(n)
a=1, b=2, f(n)=O(n)=O(n^1)
c_crit = log_2(1) = 0
f(n) = O(n^1) where 1 > 0 → Case 3
T(n) = Θ(n)   ✓
```

#### Example F: Building a Heap (heapify)
```
T(n) = 2*T(n/2) + O(log n)
a=2, b=2, f(n)=O(log n)
c_crit = 1
f(n) = O(log n) = O(n^0 * log n) where 0 < 1 → Case 1
T(n) = Θ(n)   ✓  (that's why build_heap is O(n), not O(n log n))
```

---

## 4. Amortized Analysis

Sometimes a single operation is slow, but averaged over many operations it's fast.

| Data Structure | Operation | Amortized | Worst Case |
|----------------|-----------|-----------|------------|
| `std::vector` push_back | append | O(1) | O(n) when resize |
| `std::stack` push | push | O(1) | O(1) |
| Monotonic Stack | process n elements | O(n) total | O(n) per element |
| Union-Find (path compress) | find/union | O(α(n)) ≈ O(1) | O(log n) |

---

## 5. STL Container Complexities (C++)

### Sequential Containers

| Container | Access | Search | Insert (end) | Insert (mid) | Delete |
|-----------|--------|--------|--------------|--------------|--------|
| `vector` | O(1) | O(n) | O(1) amort | O(n) | O(n) |
| `deque` | O(1) | O(n) | O(1) amort | O(n) | O(n) |
| `list` | O(n) | O(n) | O(1) | O(1) | O(1) |
| `array` | O(1) | O(n) | N/A | N/A | N/A |

### Associative Containers (Sorted — Red-Black Tree internally)

| Container | Search | Insert | Delete | Notes |
|-----------|--------|--------|--------|-------|
| `set` | O(log n) | O(log n) | O(log n) | Unique, sorted |
| `multiset` | O(log n) | O(log n) | O(log n) | Duplicates allowed |
| `map` | O(log n) | O(log n) | O(log n) | Key-value, sorted |
| `multimap` | O(log n) | O(log n) | O(log n) | Duplicate keys |

### Unordered Containers (Hash Table internally)

| Container | Search | Insert | Delete | Worst Case |
|-----------|--------|--------|--------|------------|
| `unordered_set` | O(1) avg | O(1) avg | O(1) avg | O(n) |
| `unordered_map` | O(1) avg | O(1) avg | O(1) avg | O(n) |

> **When worst case hits**: Hash collision attacks. In competitive programming, use custom hash or `std::map` when keys are adversarial (e.g., Codeforces hacks).

### Adaptors

| Container | Top/Peek | Push | Pop |
|-----------|----------|------|-----|
| `stack` (deque base) | O(1) | O(1) | O(1) |
| `queue` (deque base) | O(1) | O(1) | O(1) |
| `priority_queue` (heap) | O(1) | O(log n) | O(log n) |

### Algorithms

| Algorithm | Time | Notes |
|-----------|------|-------|
| `std::sort` | O(n log n) | Introsort (quicksort + heapsort + insertion) |
| `std::stable_sort` | O(n log² n) or O(n log n) with buffer | Merge sort based |
| `std::binary_search` | O(log n) | Requires sorted range |
| `std::lower_bound` | O(log n) | Requires sorted range |
| `std::upper_bound` | O(log n) | Requires sorted range |
| `std::nth_element` | O(n) avg | Quickselect |
| `std::make_heap` | O(n) | Linear heap build |
| `std::push_heap` | O(log n) | |
| `std::pop_heap` | O(log n) | |

---

## 6. Common Patterns and Their Complexities

| Pattern | Time | Space |
|---------|------|-------|
| Brute force (all pairs) | O(n²) | O(1) |
| Two pointers | O(n) | O(1) |
| Sliding window | O(n) | O(k) |
| Binary search | O(log n) | O(1) |
| BFS/DFS on graph (V vertices, E edges) | O(V + E) | O(V) |
| Dijkstra (priority queue) | O((V + E) log V) | O(V) |
| Dynamic programming (1D) | O(n) to O(n²) | O(n) |
| Dynamic programming (2D) | O(n * m) | O(n * m) |
| Backtracking (all subsets) | O(2ⁿ) | O(n) stack |
| Backtracking (all permutations) | O(n!) | O(n) stack |

---

## 7. Useful Math for Complexity

```
Sum of 1 to n:          n*(n+1)/2         ≈ O(n²)
Sum of log(1..n):       Θ(n log n)
Number of subsets:      2^n
Number of permutations: n!
Number of pairs:        n*(n-1)/2
log_a(n) = log_b(n) / log_b(a)           (change of base)
log_2(10^9) ≈ 30                          (memory: 10^9 ~ 2^30)
log_2(10^18) ≈ 60                         (long long max)
```

---

*Links: [[00_Home]] · [[01_Arrays_and_Hashing/00_Index|Arrays & Hashing →]]*
