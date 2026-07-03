---
tags: [dsa, dp, partition-dp, mcm]
links: ["[[07_Partition_DP_Index]]", "[[Partition_DP_Burst_Balloons]]"]
---

# Partition DP -- Matrix Chain Multiplication (MCM)

*<- [[07_Partition_DP_Index|Partition Index]] · [[Partition_DP_Burst_Balloons|Burst Balloons ->]]*

---

## What is Partition DP?
Partition DP is used when we need to find the optimal way to divide a range `[i, j]` into sub-segments. 

### The Core Pattern:
To solve for a range `[i, j]`:
1. Loop a split pointer `k` from `i` to `j-1`.
2. Divide the range into two subproblems: `[i, k]` and `[k+1, j]`.
3. Add the cost of merging these two subproblems.
4. Take the minimum (or maximum) over all possible split positions `k`.

```
Range: [ i ......................... j ]
              Split pointer k
       [ i ...... k ] [ k+1 ........ j ]
```

- **Recurrence Relation**: `solve(i, j) = min(solve(i, k) + solve(k+1, j) + mergeCost(i, k, j))` for all $k$ from $i$ to $j-1$.
- **Base Case**: `i >= j` (range of size 1 or 0 has 0 partition cost).

---

## Matrix Chain Multiplication (MCM)

**The Problem**: Given a sequence of matrices, find the most efficient way to multiply these matrices together (i.e. minimize total multiplication operations).
- You are given an array `arr` where matrix $A_i$ has dimensions `arr[i-1] x arr[i]`.

### State Formulation
Let `dp[i][j]` be the minimum operations to multiply matrices from index `i` to `j`.
- If we split at index `k`:
  - Left product dimensions: `arr[i-1] x arr[k]`
  - Right product dimensions: `arr[k] x arr[j]`
  - Cost to multiply the two final matrices: `arr[i-1] * arr[k] * arr[j]`
- **Recurrence**: `dp[i][j] = min(dp[i][k] + dp[k+1][j] + arr[i-1]*arr[k]*arr[j])` for $k \in [i, j-1]$.

---

### A. Memoization (Top-Down)
- **Time Complexity**: $O(n^3)$ (there are $n^2$ states, and each state loops $n$ times).
- **Space Complexity**: $O(n^2)$ memo table + $O(n)$ recursion stack.

```cpp
#include <vector>
#include <algorithm>

int solveMCM(int i, int j, const vector<int>& arr, vector<vector<int>>& memo) {
    if (i >= j) return 0; // base case: 1 matrix left, no multiplication cost
    if (memo[i][j] != -1) return memo[i][j];

    int minOps = 1e9;
    for (int k = i; k < j; k++) {
        int ops = solveMCM(i, k, arr, memo) + 
                  solveMCM(k + 1, j, arr, memo) + 
                  arr[i - 1] * arr[k] * arr[j];
        minOps = min(minOps, ops);
    }
    return memo[i][j] = minOps;
}

int matrixMultiplication(vector<int>& arr) {
    int n = arr.size();
    vector<vector<int>> memo(n, vector<int>(n, -1));
    return solveMCM(1, n - 1, arr, memo);
}
```

---

### B. Tabulation (Bottom-Up)
- To fill bottom-up, we must loop the range length `len` from 2 to $n-1$, and then range start `i` from 1 to $n - len$.
- **Time Complexity**: $O(n^3)$.
- **Space Complexity**: $O(n^2)$.

```cpp
#include <vector>
#include <algorithm>

int matrixMultiplicationTabulation(vector<int>& arr) {
    int n = arr.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));

    // Loop by length of chain
    for (int len = 2; len < n; len++) {
        for (int i = 1; i < n - len + 1; i++) {
            int j = i + len - 1;
            dp[i][j] = 1e9;
            for (int k = i; k < j; k++) {
                int ops = dp[i][k] + dp[k + 1][j] + arr[i - 1] * arr[k] * arr[j];
                dp[i][j] = min(dp[i][j], ops);
            }
        }
    }
    return dp[1][n - 1];
}
```

---

*<- [[07_Partition_DP_Index|Partition Index]] · [[Partition_DP_Burst_Balloons|Burst Balloons ->]]*
