---
tags: [dsa, dp, partition-dp, burst-balloons]
links: ["[[07_Partition_DP_Index]]", "[[Partition_DP_MCM]]", "[[Partition_DP_Strange_Printer]]"]
---

# Partition DP -- Burst Balloons (LC 312)

*<- [[Partition_DP_MCM|MCM]] · [[Partition_DP_Strange_Printer|Strange Printer ->]]*

---

## The Problem
You are given $n$ balloons, indexed from `0` to `n-1`. Each balloon is painted with a number on it represented by an array `nums`. You are asked to burst all the balloons.
If you burst the $i$-th balloon, you will get `nums[i-1] * nums[i] * nums[i+1]` coins. If `i-1` or `i+1` goes out of bounds, treat it as a balloon painted with a `1`.
- Return the maximum coins you can collect by bursting the balloons wisely.

---

## Why Tricky: Dependent Subproblems
In standard Partition DP (like MCM), subproblems `[i, k]` and `[k+1, j]` are completely independent.
- If we try to define our partition choice as: **"Which balloon `k` do we burst FIRST in range `[i, j]`?"**
  - Bursting `k` makes `k-1` and `k+1` adjacent!
  - This cross-boundary dependency breaks the DP optimal substructure property (the left and right subproblems are no longer independent).

---

## The Solution: Inverse Thinking (The "Last Burst" Trick)

Instead of choosing which balloon to burst first, choose **which balloon `k` is burst LAST in range `[i, j]`**.

If balloon `k` is the last to be burst in `[i, j]`:
1. All other balloons in `[i, k-1]` have already been burst.
2. All other balloons in `[k+1, j]` have already been burst.
3. Because they are all gone, the only remaining balloons adjacent to `k` at the time of its burst are the boundaries **`nums[i-1]`** (on the left) and **`nums[j+1]`** (on the right).
4. Therefore, the cost of bursting `k` at the very end is: `nums[i-1] * nums[k] * nums[j+1]`.

This keeps the subproblems `[i, k-1]` and `[k+1, j]` completely independent!

### Recurrence Relation
`solve(i, j) = max(solve(i, k-1) + solve(k+1, j) + nums[i-1]*nums[k]*nums[j+1])` for all $k \in [i, j]$.

---

## C++ Implementation (Memoization)

```cpp
#include <vector>
#include <algorithm>

int solveBalloons(int i, int j, const vector<int>& nums, vector<vector<int>>& memo) {
    if (i > j) return 0;
    if (memo[i][j] != -1) return memo[i][j];

    int maxCoins = 0;
    // Try placing the last burst balloon at every possible position k
    for (int k = i; k <= j; k++) {
        int coins = solveBalloons(i, k - 1, nums, memo) + 
                    solveBalloons(k + 1, j, nums, memo) + 
                    nums[i - 1] * nums[k] * nums[j + 1];
        maxCoins = max(maxCoins, coins);
    }
    return memo[i][j] = maxCoins;
}

int maxCoins(vector<int>& nums) {
    int n = nums.size();
    // Pad the array boundaries with 1s as requested
    vector<int> paddedNums(n + 2, 1);
    for (int i = 0; i < n; i++) paddedNums[i + 1] = nums[i];

    vector<vector<int>> memo(n + 2, vector<int>(n + 2, -1));
    return solveBalloons(1, n, paddedNums, memo);
}
// Time Complexity: O(n^3)
// Space Complexity: O(n^2)
```

---

*<- [[Partition_DP_MCM|MCM]] · [[Partition_DP_Strange_Printer|Strange Printer ->]]*
