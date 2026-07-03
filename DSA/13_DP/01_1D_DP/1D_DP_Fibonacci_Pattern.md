---
tags: [dsa, dp, 1d-dp, fibonacci, climbing-stairs, house-robber]
links: ["[[01_1D_DP_Index]]", "[[../../00_Complexity_Cheatsheet]]"]
---

# 1D DP -- Fibonacci & Step Patterns

*<- [[01_1D_DP_Index|1D DP Index]] · [[1D_DP_Coin_Change|Coin Change ->]]*

---

## 1. Fibonacci Sequence

**The Problem**: Find the $n$-th Fibonacci number: $F(n) = F(n-1) + F(n-2)$, with base cases $F(0) = 0, F(1) = 1$.

### A. Trivial Recursive Approach
- **Core Intuition**: Direct implementation of the recurrence relation.
- **Recursion Tree for n = 4**:
```
                       F(4)
                      /    \
                  F(3)      F(2)
                 /    \     /   \
               F(2)   F(1) F(1)  F(0)
              /    \
            F(1)   F(0)
```
Notice that $F(2)$ is computed twice. As $n$ grows, this duplication grows exponentially.
- **Time Complexity**: $O(2^n)$ (see [[../../00_Complexity_Cheatsheet#Big-O]] for tree recursion math).
- **Space Complexity**: $O(n)$ recursion stack.

```cpp
int fibRecursive(int n) {
    if (n <= 1) return n;
    return fibRecursive(n - 1) + fibRecursive(n - 2);
}
```

---

### B. Top-Down Dynamic Programming (Memoization)
- **Core Intuition**: Store computed states in an array (cache) to avoid redundant recursive computations.
- **State Definition**: `memo[i]` stores the $i$-th Fibonacci number.
- **Time Complexity**: $O(n)$ (each state computed exactly once).
- **Space Complexity**: $O(n)$ memo array + $O(n)$ recursion stack.

```cpp
#include <vector>

int fibMemoHelper(int n, vector<int>& memo) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n]; // return cached result

    return memo[n] = fibMemoHelper(n - 1, memo) + fibMemoHelper(n - 2, memo);
}

int fibMemo(int n) {
    vector<int> memo(n + 1, -1);
    return fibMemoHelper(n, memo);
}
```

---

### C. Bottom-Up Dynamic Programming (Tabulation)
- **Core Intuition**: Eliminate recursion overhead by filling the DP table iteratively from base cases up.
- **DP Table definition**: `dp[i]` is the $i$-th Fibonacci number.
- **Time Complexity**: $O(n)$.
- **Space Complexity**: $O(n)$ to store the DP array.

```cpp
#include <vector>

int fibTabulation(int n) {
    if (n <= 1) return n;
    vector<int> dp(n + 1);
    dp[0] = 0;
    dp[1] = 1;

    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

---

### D. Space-Optimized Tabulation
- **Core Intuition**: To calculate `dp[i]`, we only ever need `dp[i-1]` and `dp[i-2]`. We don't need to store the entire array of size $n$; just track the last two values.
- **Time Complexity**: $O(n)$.
- **Space Complexity**: $O(1)$.

```cpp
int fibSpaceOptimized(int n) {
    if (n <= 1) return n;
    int prev2 = 0; // F(i-2)
    int prev1 = 1; // F(i-1)
    int curr = 0;

    for (int i = 2; i <= n; i++) {
        curr = prev1 + prev2;
        prev2 = prev1; // shift window forward
        prev1 = curr;
    }
    return curr;
}
```

### Dry Run Variable State Table:
For `n = 4`:
| Iteration (i) | `prev2` | `prev1` | `curr` |
|---|---|---|---|
| Initial | 0 | 1 | 0 |
| i = 2 | 1 | 1 | 1 |
| i = 3 | 1 | 2 | 2 |
| i = 4 | 2 | 3 | 3 |
- **Result**: `3` (Correct).

---

## 2. Climbing Stairs (LC 70)

**The Problem**: You are climbing a staircase. It takes $n$ steps to reach the top. Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

### State Formulation
Let `dp[i]` be the number of ways to reach step `i`.
- To reach step `i`, you must come from either step `i-1` (by taking 1 step) or step `i-2` (by taking 2 steps).
- **Recurrence Relation**: `dp[i] = dp[i-1] + dp[i-2]`
- **Base cases**: `dp[1] = 1` (1 way), `dp[2] = 2` (2 ways: 1+1 or 2).

This is mathematically identical to the Fibonacci sequence, shifted by 1.

```cpp
int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1; // ways(1)
    int prev1 = 2; // ways(2)
    int curr = 0;

    for (int i = 3; i <= n; i++) {
        curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return curr;
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

## 3. House Robber (LC 198)

**The Problem**: You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed. The only constraint stopping you from robbing each of them is that adjacent houses have security systems connected and **it will automatically contact the police if two adjacent houses were broken into on the same night**. Given an array of integers representing the amount of money of each house, return the maximum money you can rob tonight without alerting the police.

### State Formulation
Let `dp[i]` be the maximum money you can rob from the first `i` houses.
For house `i`, you have two choices:
1. **Rob house `i`**: If you rob house `i`, you cannot rob house `i-1`. Your total money will be `nums[i] + dp[i-2]`.
2. **Do not rob house `i`**: You can safely take the money from the previous house, i.e. `dp[i-1]`.

- **Recurrence Relation**: `dp[i] = max(dp[i-1], nums[i] + dp[i-2])`
- **Base cases**:
  - `dp[0] = nums[0]` (only one house to rob)
  - `dp[1] = max(nums[0], nums[1])` (pick the one with more money)

### Space-Optimized Code

```cpp
#include <vector>
#include <algorithm>

int rob(vector<int>& nums) {
    int n = nums.size();
    if (n == 1) return nums[0];

    int prev2 = nums[0]; // dp[i-2]
    int prev1 = max(nums[0], nums[1]); // dp[i-1]
    int curr = prev1;

    for (int i = 2; i < n; i++) {
        curr = max(prev1, nums[i] + prev2);
        prev2 = prev1;
        prev1 = curr;
    }
    return curr;
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

### Dry Run Variable State Table:
For `nums = [2, 7, 9, 3, 1]`:
| Iteration (i) | `nums[i]` | `prev2` | `prev1` | `curr` | Choice Formula |
|---|---|---|---|---|---|
| Initial | - | 2 | 7 | 7 | Base Cases resolved |
| i = 2 | 9 | 7 | 7 | 11 | `max(7, 9 + 2)` |
| i = 3 | 3 | 7 | 11 | 11 | `max(11, 3 + 7)` |
| i = 4 | 1 | 11 | 11 | 12 | `max(11, 1 + 11)` |
- **Result**: `12` (Correct: Rob house 0, 2, 4 -> 2 + 9 + 1 = 12).

---

*<- [[01_1D_DP_Index|1D DP Index]] · [[1D_DP_Coin_Change|Coin Change ->]]*
