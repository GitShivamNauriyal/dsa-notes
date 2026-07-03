---
tags: [dsa, binary-search, tricky, hard, interview, infinite-array]
links: ["[[05_Binary_Search_Index]]", "[[05_Binary_Search_Problems_and_Exercises]]", "[[../06_Linked_List/06_Linked_List_Index]]"]
---

# Binary Search -- Tricky & Higher-Order

*<- [[05_Binary_Search_Problems_and_Exercises|Problems]] · [[../06_Linked_List/06_Linked_List_Index|Linked List ->]]*

---

## 1. Binary Search on an Infinite Array

**Why tricky**: No `right` boundary exists. You can't set `right = n-1` because you don't know n.

**Strategy**: Exponential expansion to find a range `[left, right]` that contains the target, then do standard binary search in that range. Double the right boundary each time.

```cpp
// Search in an "infinite sorted array" (accessed via get(i) API)
// Assume get(i) returns INT_MAX if out of bounds

int searchInfinite(vector<int>& nums, int target) {
    // Phase 1: find bounds by exponential expansion
    int left = 0, right = 1;
    // Double the right boundary until we find a range containing target
    while (nums[right] < target) {
        left = right;
        right *= 2;  // exponential growth: 1,2,4,8,16,...
        // This finds the range in O(log n) steps where n = position of target
    }

    // Phase 2: standard binary search within [left, right]
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
// Time: O(log n) where n is the index of target
// Key insight: if target is at index n, we find the range in O(log n),
// then search the range in O(log n). Total: O(log n).
```

---

## 2. Fractional / Real-Valued Binary Search

**When**: The answer is a real number (float), not an integer. Cannot use integer mid tricks.  
**Technique**: Run for a fixed number of iterations (100 iterations gives precision of 2^-100).

```cpp
// Find x such that f(x) = target (f is monotonic)
// Example: find the nth root of m with precision 1e-7

double nthRoot(int n, double m) {
    double lo = 0, hi = max(1.0, m);  // for m<1, root > m, so hi = 1 is enough? No.
    // Actually for m >= 1: root is in [1, m]. For m < 1: root is in [m, 1].
    if (m < 1) { lo = m; hi = 1.0; }

    for (int iter = 0; iter < 200; iter++) {  // 200 iterations for high precision
        double mid = lo + (hi - lo) / 2.0;
        double val = 1.0;
        for (int i = 0; i < n; i++) val *= mid;  // mid^n

        if (val < m) lo = mid;
        else hi = mid;
    }
    return lo;
}

// LC 774 -- Minimize Max Distance to Gas Station
// Answer is a real number: smallest max gap after adding k stations
double minMaxGasDist(vector<int>& stations, int k) {
    double lo = 0, hi = stations.back() - stations.front();

    auto countNeeded = [&](double d) -> int {
        int count = 0;
        for (int i = 1; i < (int)stations.size(); i++)
            count += (int)((stations[i] - stations[i-1]) / d);
            // how many stations needed in this gap to keep spacing <= d
        return count;
    };

    for (int iter = 0; iter < 100; iter++) {
        double mid = lo + (hi - lo) / 2.0;
        if (countNeeded(mid) > k) lo = mid;
        else hi = mid;
    }
    return lo;
}
// Time: O(100 * n) = O(n), Space: O(1)
```

---

## 3. Binary Search on a 2D Matrix

**Two types**:
- **Type 1** (LC 74): Matrix where each row is sorted AND first element of row i+1 > last of row i. Treat as flat 1D array.
- **Type 2** (LC 240): Sorted rows and columns but NO guarantee first of next row > last of current. Cannot flatten. Use staircase search instead.

```cpp
// LC 74 -- Search a 2D Matrix (Type 1: can flatten)
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int rows = matrix.size(), cols = matrix[0].size();
    int left = 0, right = rows * cols - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        // Convert 1D index to 2D
        int val = matrix[mid / cols][mid % cols];
        if (val == target) return true;
        else if (val < target) left = mid + 1;
        else right = mid - 1;
    }
    return false;
}
// Time: O(log(rows*cols)), Space: O(1)

// LC 240 -- Search a 2D Matrix II (Type 2: staircase search)
// Start from top-right corner. Go left if too big, go down if too small.
bool searchMatrixII(vector<vector<int>>& matrix, int target) {
    int r = 0, c = (int)matrix[0].size() - 1;
    while (r < (int)matrix.size() && c >= 0) {
        if (matrix[r][c] == target) return true;
        else if (matrix[r][c] > target) c--;  // current too big, go left
        else r++;                              // current too small, go down
    }
    return false;
}
// Time: O(rows + cols), Space: O(1)
// Why O(rows + cols): each step eliminates a row or column
```

---

## 4. Parallel Binary Search

**When**: Multiple independent binary search queries on the same array. Instead of doing each separately in O(Q log n * check_cost), do all Q in parallel reducing total checks.

**Technique**: Run log(n) rounds. In each round, compute mid for all active queries. Run one pass to answer all mids simultaneously. Then update each query's range.

```cpp
// Find for each query[i] = {l, r, target}: the leftmost index in nums[l..r] >= target
// Instead of Q separate binary searches (O(Q log n)), do them together.
// Useful when the "check" function is expensive (e.g., involves segment tree query).

void parallelBinarySearch(
    vector<int>& nums,
    vector<tuple<int,int,int>>& queries,  // {l, r, target}
    vector<int>& answers
) {
    int Q = queries.size(), n = nums.size();
    // Each query tracks its [lo, hi] binary search range
    vector<int> lo(Q, 0), hi(Q, n), mid(Q);
    answers.assign(Q, n);

    for (int iter = 0; iter < 20; iter++) {  // log2(n) rounds
        // Group queries by their current mid
        vector<vector<int>> at(n + 1);
        bool anyActive = false;
        for (int q = 0; q < Q; q++) {
            if (lo[q] < hi[q]) {
                mid[q] = lo[q] + (hi[q] - lo[q]) / 2;
                at[mid[q]].push_back(q);
                anyActive = true;
            }
        }
        if (!anyActive) break;

        // One pass through nums to answer all queries at their current mid
        for (int i = 0; i < n; i++) {
            for (int q : at[i]) {
                auto [l, r, target] = queries[q];
                if (nums[i] >= target) {
                    // First valid position is at most i
                    answers[q] = i;
                    hi[q] = mid[q];
                } else {
                    lo[q] = mid[q] + 1;
                }
            }
        }
    }
}
// Time: O((n + Q) * log n) vs O(Q * n) naive or O(Q * log n * check) sequential
```

---

## 5. Binary Search with Monotonic Function -- Finding Zeros

**When**: You have a strictly decreasing function f(x) and want to find where f(x) = 0 (or crosses a threshold). f changes sign: positive for small x, negative for large x.

```cpp
// Find x in [lo, hi] where f(x) transitions from positive to negative
// (e.g., profit = revenue - cost, find break-even point)

int findBreakEven(function<int(int)> f, int lo, int hi) {
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (f(mid) > 0) lo = mid + 1;  // still positive, look right
        else hi = mid;                  // non-positive, could be answer
    }
    return lo;
}
```

---

## 6. Rotated Array with Duplicates (LC 81 and LC 154)

**Why tricky**: With duplicates, you cannot determine which half is sorted when `nums[left] == nums[mid]`. Only safe move: `left++` (linear shrink). Worst case degrades to O(n).

```cpp
// LC 81 -- Search in Rotated Sorted Array II (with duplicates)
bool searchWithDups(vector<int>& nums, int target) {
    int left = 0, right = (int)nums.size() - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return true;

        // Cannot determine sorted half when nums[left] == nums[mid]
        if (nums[left] == nums[mid]) {
            left++;  // shrink left by 1 (safe, but could be O(n) worst case)
            continue;
        }

        if (nums[left] <= nums[mid]) {
            if (target >= nums[left] && target < nums[mid]) right = mid - 1;
            else left = mid + 1;
        } else {
            if (target > nums[mid] && target <= nums[right]) left = mid + 1;
            else right = mid - 1;
        }
    }
    return false;
}
// Time: O(log n) average, O(n) worst case (all duplicates)
```

---

## 7. K-th Smallest Element in Sorted Matrix (LC 378)

**Why tricky**: The matrix is sorted row-wise and column-wise. Binary search on the VALUE (not index). Count how many elements are <= mid.

```cpp
// Count elements <= val in a sorted matrix
int countLE(vector<vector<int>>& matrix, int val) {
    int n = matrix.size(), count = 0;
    int r = n - 1, c = 0;  // start from bottom-left
    while (r >= 0 && c < n) {
        if (matrix[r][c] <= val) {
            count += r + 1;  // entire column up to r is <= val
            c++;
        } else {
            r--;
        }
    }
    return count;
}

int kthSmallest(vector<vector<int>>& matrix, int k) {
    int n = matrix.size();
    int lo = matrix[0][0], hi = matrix[n-1][n-1];
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        // count elements <= mid. If >= k, answer is <= mid.
        if (countLE(matrix, mid) >= k) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
// Time: O(n log(max-min)), Space: O(1)
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Search in Infinite Sorted Array | Medium | [GFG](https://www.geeksforgeeks.org/find-position-element-sorted-array-infinite-numbers/) |
| 2 | Search a 2D Matrix | Medium | [LC 74](https://leetcode.com/problems/search-a-2d-matrix/) |
| 3 | Search a 2D Matrix II | Medium | [LC 240](https://leetcode.com/problems/search-a-2d-matrix-ii/) |
| 4 | Kth Smallest Element in Sorted Matrix | Medium | [LC 378](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/) |
| 5 | Search in Rotated Array II | Medium | [LC 81](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) |
| 6 | Find Minimum in Rotated Array II | Hard | [LC 154](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/) |
| 7 | Minimize Max Distance to Gas Station | Hard | [LC 774](https://leetcode.com/problems/minimize-max-distance-to-gas-station/) |
| 8 | Nth Root of M (fractional) | Medium | [GFG](https://www.geeksforgeeks.org/n-th-root-of-a-number/) |
| 9 | Kth Smallest in Multiplication Table | Hard | [LC 668](https://leetcode.com/problems/kth-smallest-number-in-multiplication-table/) |
| 10 | Smallest Range Covering All Lists | Hard | [LC 632](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/) |

---

*<- [[05_Binary_Search_Problems_and_Exercises|Problems]] · [[../06_Linked_List/06_Linked_List_Index|Linked List ->]]*
