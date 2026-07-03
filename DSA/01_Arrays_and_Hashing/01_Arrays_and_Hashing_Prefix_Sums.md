---
tags: [dsa, arrays, prefix-sum, difference-array]
links: ["[[01_Arrays_and_Hashing_Index]]", "[[01_Arrays_and_Hashing_Arrays_Basics]]", "[[01_Arrays_and_Hashing_Hashing]]"]
---

# Prefix Sums & Difference Arrays

*← [[01_Arrays_and_Hashing_Arrays_Basics|Arrays Basics]] · [[01_Arrays_and_Hashing_Hashing|Hashing →]]*

---

## The Core Problem

You have an array and someone asks: "What is the sum of elements from index `l` to index `r`?"

**Naive approach**: loop from l to r every time → O(n) per query.  
If there are Q queries, total time is O(n * Q) — too slow.

**Prefix sum approach**: precompute in O(n), answer every query in O(1).

---

## Pattern 1: 1D Prefix Sum (Range Sum Query)

**Why it works**: `sum(l, r) = sum(0, r) - sum(0, l-1)`.  
If we pre-store the cumulative sum at every index, subtraction gives us any range instantly.

```cpp
#include <vector>

// Build prefix sum array
// prefix[i] = sum of elements from index 0 to i-1
// (1-indexed prefix for cleaner code — prefix[0] = 0)
vector<int> buildPrefix(const vector<int>& arr) {
    int n = arr.size();
    vector<int> prefix(n + 1, 0);
    for (int i = 0; i < n; i++) {
        prefix[i + 1] = prefix[i] + arr[i];
    }
    return prefix;
}

// Query: sum from index l to r (0-indexed, inclusive)
int rangeSum(const vector<int>& prefix, int l, int r) {
    return prefix[r + 1] - prefix[l];
    // prefix[r+1] = sum(0..r), prefix[l] = sum(0..l-1)
    // difference = sum(l..r)
}

// Example:
// arr =    [1, 2, 3, 4, 5]
// prefix = [0, 1, 3, 6, 10, 15]
// sum(1,3) = prefix[4] - prefix[1] = 10 - 1 = 9  ✓ (2+3+4=9)
```

**Time**: O(n) build, O(1) query  
**Space**: O(n)

---

## Pattern 2: Prefix Sum for Counting (Subarray Sum = K)

**Why**: Instead of sum of values, prefix sums can count subarrays satisfying a condition.  
Key insight: if `prefix[j] - prefix[i] = k`, then subarray `[i+1..j]` has sum k.  
Rearranging: `prefix[i] = prefix[j] - k`.

```cpp
#include <vector>
#include <unordered_map>

// Count subarrays with sum equal to k
// LeetCode 560 — Subarray Sum Equals K
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int, int> freq;  // freq[prefix_sum] = count
    freq[0] = 1;   // empty prefix (sum 0 seen once, before we start)
    
    int prefixSum = 0;
    int count = 0;
    
    for (int x : nums) {
        prefixSum += x;
        // Check: how many previous prefix sums equal (prefixSum - k)?
        // If prefix[j] - prefix[i] = k  →  prefix[i] = prefix[j] - k
        count += freq[prefixSum - k];
        freq[prefixSum]++;
    }
    return count;
}
// Time: O(n), Space: O(n)
```

---

## Pattern 3: 2D Prefix Sum (Matrix Range Query)

**Why**: Same idea extended to 2D. Pre-compute cumulative sum of entire top-left rectangle for any cell.

```
prefix[i][j] = sum of all elements in rectangle (0,0) to (i-1, j-1)

Query sum of rectangle (r1,c1) to (r2,c2):
= prefix[r2+1][c2+1]
  - prefix[r1][c2+1]    (remove top strip)
  - prefix[r2+1][c1]    (remove left strip)
  + prefix[r1][c1]      (add back the corner subtracted twice)
```

```
Inclusion-Exclusion visualization:
┌─────────────────────────┐
│         TOP             │
│   ┌─────────────────┐   │
│   │                 │   │
│   │    QUERY RECT   │   │
│   │                 │   │
│   └─────────────────┘   │
└─────────────────────────┘

Answer = Big - Top - Left + Corner
```

```cpp
#include <vector>

struct Matrix2DPrefix {
    vector<vector<int>> prefix;
    int rows, cols;

    // Build: O(rows * cols)
    Matrix2DPrefix(const vector<vector<int>>& grid) {
        rows = grid.size();
        cols = grid[0].size();
        prefix.assign(rows + 1, vector<int>(cols + 1, 0));
        
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                prefix[i][j] = grid[i-1][j-1]
                              + prefix[i-1][j]    // top
                              + prefix[i][j-1]    // left
                              - prefix[i-1][j-1]; // corner (added twice)
            }
        }
    }

    // Query: sum in rectangle (r1,c1) to (r2,c2), 0-indexed — O(1)
    int query(int r1, int c1, int r2, int c2) {
        return prefix[r2+1][c2+1]
             - prefix[r1][c2+1]
             - prefix[r2+1][c1]
             + prefix[r1][c1];
    }
};
```

**Time**: O(n*m) build, O(1) query  
**Space**: O(n*m)

---

## Pattern 4: Difference Array (Range Update in O(1))

**Problem flip**: Instead of many range queries on a fixed array, you now need many **range updates** (add `v` to all elements from `l` to `r`), then read the final array.

**Naive**: loop from l to r for each update → O(n) per update.  
**Difference array**: O(1) per update, O(n) to reconstruct.

**Why it works**: The difference array `diff[i] = arr[i] - arr[i-1]`.  
Adding `v` to `arr[l..r]` means:
- `diff[l] += v`  (sum from this point increases by v)
- `diff[r+1] -= v`  (sum stops increasing after r)
Then reconstruct `arr` using prefix sum of `diff`.

```cpp
#include <vector>

// Range update: add val to arr[l..r] for multiple queries
// Then get final array
vector<int> rangeUpdateAndQuery(
    int n,
    vector<tuple<int,int,int>>& updates  // {l, r, val}
) {
    vector<int> diff(n + 1, 0);  // difference array, 1 extra for safety

    for (auto& [l, r, val] : updates) {  // C++17 structured binding
        diff[l] += val;
        if (r + 1 <= n - 1) diff[r + 1] -= val;
    }

    // Reconstruct by taking prefix sum of diff
    vector<int> result(n, 0);
    int running = 0;
    for (int i = 0; i < n; i++) {
        running += diff[i];
        result[i] = running;
    }
    return result;
}

// Example:
// n=5, add 3 to [1..3], add 2 to [0..2]
// diff after updates: [2, 3, 0, -3, -2]
//   (add 2 at 0, subtract at 3; add 3 at 1, subtract at 4)
// prefix sum: [2, 5, 5, 2, 0]
// final array: [2, 5, 5, 2, 0] ✓
```

**Time**: O(1) per update, O(n) to reconstruct  
**Space**: O(n)

---

## Pattern 5: Prefix XOR (Subarray XOR Queries)

**Why**: XOR has a special property: `a XOR a = 0`. So `prefix[r] XOR prefix[l-1] = XOR of subarray [l..r]`.

```cpp
#include <vector>

vector<int> buildXorPrefix(const vector<int>& arr) {
    int n = arr.size();
    vector<int> prefix(n + 1, 0);
    for (int i = 0; i < n; i++) {
        prefix[i + 1] = prefix[i] ^ arr[i];
    }
    return prefix;
}

int xorQuery(const vector<int>& prefix, int l, int r) {
    return prefix[r + 1] ^ prefix[l];
}
// If prefix[l] = XOR(0..l-1), prefix[r+1] = XOR(0..r),
// then prefix[r+1] ^ prefix[l] = XOR(l..r)  ← because common prefix cancels
```

---

## Examples

### Example 1 — LC 303: Range Sum Query (Easy)

> Given an integer array, handle multiple queries `sumRange(left, right)`.

```cpp
class NumArray {
    vector<int> prefix;
public:
    NumArray(vector<int>& nums) {
        int n = nums.size();
        prefix.resize(n + 1, 0);
        for (int i = 0; i < n; i++)
            prefix[i + 1] = prefix[i] + nums[i];
    }
    int sumRange(int left, int right) {
        return prefix[right + 1] - prefix[left];
    }
};
// Time: O(n) build, O(1) per query
```

### Example 2 — LC 304: Range Sum Query 2D (Medium)

> Matrix range sum queries.

```cpp
class NumMatrix {
    vector<vector<int>> prefix;
public:
    NumMatrix(vector<vector<int>>& matrix) {
        int r = matrix.size(), c = matrix[0].size();
        prefix.assign(r + 1, vector<int>(c + 1, 0));
        for (int i = 1; i <= r; i++)
            for (int j = 1; j <= c; j++)
                prefix[i][j] = matrix[i-1][j-1]
                              + prefix[i-1][j]
                              + prefix[i][j-1]
                              - prefix[i-1][j-1];
    }
    int sumRegion(int r1, int c1, int r2, int c2) {
        return prefix[r2+1][c2+1]
             - prefix[r1][c2+1]
             - prefix[r2+1][c1]
             + prefix[r1][c1];
    }
};
```

### Example 3 — LC 560: Subarray Sum Equals K (Medium)

Already shown in Pattern 2. Key idea: `freq[prefixSum - k]` gives how many subarrays ending here have sum k.

### Example 4 — LC 1109: Corporate Flight Bookings (Medium)

> `n` flights, `k` bookings each as `[first, last, seats]`. Return seats booked on each flight.

This is a direct application of **difference array**.

```cpp
vector<int> corpFlightBookings(
    vector<vector<int>>& bookings, int n
) {
    vector<int> diff(n + 2, 0);
    for (auto& b : bookings) {
        int first = b[0], last = b[1], seats = b[2];
        diff[first] += seats;
        diff[last + 1] -= seats;
    }
    vector<int> ans(n, 0);
    int running = 0;
    for (int i = 1; i <= n; i++) {
        running += diff[i];
        ans[i - 1] = running;
    }
    return ans;
}
```

---

## Exercises

| # | Problem | Platform | Difficulty | Pattern |
|---|---------|----------|------------|---------|
| 1 | [303. Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/) | LeetCode | 🟢 Easy | 1D Prefix |
| 2 | [304. Range Sum Query 2D](https://leetcode.com/problems/range-sum-query-2d-immutable/) | LeetCode | 🟡 Medium | 2D Prefix |
| 3 | [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | LeetCode | 🟡 Medium | Prefix + HashMap |
| 4 | [1109. Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/) | LeetCode | 🟡 Medium | Difference Array |
| 5 | [1094. Car Pooling](https://leetcode.com/problems/car-pooling/) | LeetCode | 🟡 Medium | Difference Array |
| 6 | [525. Contiguous Array](https://leetcode.com/problems/contiguous-array/) | LeetCode | 🟡 Medium | Prefix + HashMap (treat 0 as -1) |
| 7 | [1480. Running Sum of 1d Array](https://leetcode.com/problems/running-sum-of-1d-array/) | LeetCode | 🟢 Easy | Prefix basics |
| 8 | [974. Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/) | LeetCode | 🟡 Medium | Prefix + Modulo |
| 9 | [Range Update Point Query](https://codeforces.com/problemset/problem/296/C) | Codeforces | 🟡 Medium | Difference Array |

---

*← [[01_Arrays_and_Hashing_Arrays_Basics|Arrays Basics]] · [[01_Arrays_and_Hashing_Hashing|Hashing →]]*
