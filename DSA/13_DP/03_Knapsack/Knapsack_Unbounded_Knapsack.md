---
tags: [dsa, dp, knapsack, unbounded-knapsack]
links: ["[[03_Knapsack_Index]]", "[[Knapsack_01_Knapsack]]", "[[Knapsack_Subset_Sum]]"]
---

# Knapsack -- Unbounded Knapsack

*<- [[Knapsack_01_Knapsack|0/1 Knapsack]] · [[Knapsack_Subset_Sum|Subset Sum ->]]*

---

## The Problem
You are given weights and values of $n$ items. Put these items in a knapsack of capacity $W$ to get the maximum total value.
- Constraint: You have **infinite copies** of each item. You can pick any item multiple times.

---

## Recurrence Formulation
In 0/1 Knapsack, if we include item `i`, we transition to row `i-1` (meaning we cannot reuse it):
`dp[i][w] = max(dp[i-1][w], val[i] + dp[i-1][w - wt[i]])`

In **Unbounded Knapsack**, if we include item `i`, we transition to the **same row `i`** (meaning we can reuse it again):
`dp[i][w] = max(dp[i-1][w], val[i] + dp[i][w - wt[i]])`

---

## 1D Space Optimization (Forward Loop)
- In 0/1 Knapsack, we looped capacity $w$ **backwards** (from $W$ down to $wt[i]$) to ensure we used values from the previous row.
- In **Unbounded Knapsack**, we loop capacity $w$ **forwards** (from $wt[i]$ up to $W$). This allows current updates to immediately build upon updates made in the *same* step, naturally representing infinite reuse of elements!

```cpp
#include <vector>
#include <algorithm>

int unboundedKnapsack(int W, const vector<int>& wt, const vector<int>& val) {
    int n = wt.size();
    vector<int> dp(W + 1, 0);

    for (int i = 0; i < n; i++) {
        // Loop FORWARDS to allow infinite reuse
        for (int w = wt[i]; w <= W; w++) {
            dp[w] = max(dp[w], val[i] + dp[w - wt[i]]);
        }
    }
    return dp[W];
}
// Time Complexity: O(n * W)
// Space Complexity: O(W)
```

---

## Detailed Trace Comparison: 0/1 vs Unbounded
Let's see what happens to `dp` with one item: `wt = [2]`, `val = [3]`, `W = 5`.

### 0/1 Knapsack (Backward Loop):
- Initial: `[0, 0, 0, 0, 0, 0]`
- Loop `w` from 5 down to 2:
  - `w=5`: `dp[5] = max(0, 3 + dp[3]) = 3`
  - `w=4`: `dp[4] = max(0, 3 + dp[2]) = 3`
  - `w=3`: `dp[3] = max(0, 3 + dp[1]) = 3`
  - `w=2`: `dp[2] = max(0, 3 + dp[0]) = 3`
- Final Array: `[0, 0, 3, 3, 3, 3]`. Max value = `3` (Item 2 can only be picked once).

### Unbounded Knapsack (Forward Loop):
- Initial: `[0, 0, 0, 0, 0, 0]`
- Loop `w` from 2 up to 5:
  - `w=2`: `dp[2] = max(0, 3 + dp[0]) = 3`
  - `w=3`: `dp[3] = max(0, 3 + dp[1]) = 3`
  - `w=4`: `dp[4] = max(0, 3 + dp[2]) = max(0, 3 + 3) = 6` (reused!)
  - `w=5`: `dp[5] = max(0, 3 + dp[3]) = max(0, 3 + 3) = 6` (reused!)
- Final Array: `[0, 0, 3, 3, 6, 6]`. Max value = `6` (Item 2 picked twice).

---

*<- [[Knapsack_01_Knapsack|0/1 Knapsack]] · [[Knapsack_Subset_Sum|Subset Sum ->]]*
