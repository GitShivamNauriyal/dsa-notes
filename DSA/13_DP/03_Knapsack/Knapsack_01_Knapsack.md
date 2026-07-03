---
tags: [dsa, dp, knapsack, 01-knapsack]
links: ["[[03_Knapsack_Index]]", "[[Knapsack_Unbounded_Knapsack]]"]
---

# Knapsack -- 0/1 Knapsack

*<- [[03_Knapsack_Index|Knapsack Index]] · [[Knapsack_Unbounded_Knapsack|Unbounded Knapsack ->]]*

---

## The Problem
You are given weights and values of $n$ items. Put these items in a knapsack of capacity $W$ to get the maximum total value in the knapsack.
- Constraint: You cannot break items; you either pick an item (`1`) or don't (`0`).

---

## State Formulation
Let `dp[i][w]` be the maximum value obtained from the first `i` items with a remaining capacity of `w`.
For the $i$-th item (weight `wt[i]`, value `val[i]`), you have two choices:
1. **Exclude it**: The max value remains `dp[i-1][w]`.
2. **Include it** (only if `wt[i] <= w`): The max value becomes `val[i] + dp[i-1][w - wt[i]]`.

- **Recurrence Relation**: `dp[i][w] = max(dp[i-1][w], val[i] + dp[i-1][w - wt[i]])`
- **Base cases**: `dp[0][w] = 0` (no items), `dp[i][0] = 0` (capacity is 0).

---

### A. Tabulation (2D Array)
- **Time Complexity**: $O(n \times W)$.
- **Space Complexity**: $O(n \times W)$.

```cpp
#include <vector>
#include <algorithm>

int knapsack2D(int W, const vector<int>& wt, const vector<int>& val) {
    int n = wt.size();
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= n; i++) {
        for (int w = 1; w <= W; w++) {
            if (wt[i - 1] <= w) {
                dp[i][w] = max(dp[i - 1][w], val[i - 1] + dp[i - 1][w - wt[i - 1]]);
            } else {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    return dp[n][W];
}
```

---

### B. Space Optimization (1D Array)
- **Core Intuition**: To calculate the current row `i` for capacity `w`, we only need values from the previous row `i-1` at indices `w` and `w - wt[i-1]`.
- If we update a **single row array** `dp` from **right to left (backwards)** (from `W` down to `wt[i-1]`), we ensure that `dp[w - wt[i-1]]` still holds the value from the *previous* row (not yet overwritten in the current loop iteration).
- **Time Complexity**: $O(n \times W)$.
- **Space Complexity**: $O(W)$ (massive space savings!).

```cpp
#include <vector>
#include <algorithm>

int knapsack1D(int W, const vector<int>& wt, const vector<int>& val) {
    int n = wt.size();
    vector<int> dp(W + 1, 0);

    for (int i = 0; i < n; i++) {
        // Loop backwards to prevent using the same item multiple times
        for (int w = W; w >= wt[i]; w--) {
            dp[w] = max(dp[w], val[i] + dp[w - wt[i]]);
        }
    }
    return dp[W];
}
```

### Dry Run Row Transition Table:
For `wt = [2, 3, 4]`, `val = [3, 4, 5]`, `W = 5`:
- Initial dp array (size 6): `[0, 0, 0, 0, 0, 0]`

| Item (i) | wt, val | Loop w: 5 down to wt | `dp` Array State |
|---|---|---|---|
| Initial | - | - | `[0, 0, 0, 0, 0, 0]` |
| 0 | 2, 3 | 5 down to 2 | `dp[5]=max(0, 3+dp[3])=3`, `dp[4]=3`, `dp[3]=3`, `dp[2]=3`<br>Array: `[0, 0, 3, 3, 3, 3]` |
| 1 | 3, 4 | 5 down to 3 | `dp[5]=max(3, 4+dp[2])=7`, `dp[4]=max(3, 4+dp[1])=4`, `dp[3]=max(3, 4+dp[0])=4`<br>Array: `[0, 0, 3, 4, 4, 7]` |
| 2 | 4, 5 | 5 down to 4 | `dp[5]=max(7, 5+dp[1])=7`, `dp[4]=max(4, 5+dp[0])=5`<br>Array: `[0, 0, 3, 4, 5, 7]` |
- **Result**: `7` (Correct: Pick items 0 and 1 -> weight 5, val 7).

---

*<- [[03_Knapsack_Index|Knapsack Index]] · [[Knapsack_Unbounded_Knapsack|Unbounded Knapsack ->]]*
