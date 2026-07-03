---
tags: [dsa, dp, knapsack, subset-sum]
links: ["[[03_Knapsack_Index]]", "[[Knapsack_Unbounded_Knapsack]]", "[[Knapsack_Target_Sum]]"]
---

# Knapsack -- Subset Sum & Equal Partition

*<- [[Knapsack_Unbounded_Knapsack|Unbounded Knapsack]] · [[Knapsack_Target_Sum|Target Sum ->]]*

---

## 1. Subset Sum Problem

**The Problem**: Given an array of non-negative integers `arr` and a value `target`, determine if there is a subset of the given set with sum equal to `target`.

### State Formulation
Let `dp[i][j]` be a boolean indicating if a subset of the first `i` elements can sum up to `j`.
For the $i$-th element `arr[i-1]`, we have two choices:
1. **Exclude it**: The sum `j` must be formed by the first `i-1` elements. `dp[i][j] = dp[i-1][j]`
2. **Include it** (only if `arr[i-1] <= j`): The remaining sum `j - arr[i-1]` must be formed by the first `i-1` elements. `dp[i][j] = dp[i-1][j - arr[i-1]]`

- **Recurrence Relation**: `dp[i][j] = dp[i-1][j] || dp[i-1][j - arr[i-1]]`
- **Base cases**:
  - `dp[i][0] = true` (sum 0 is always possible using the empty subset)
  - `dp[0][j] = false` for $j > 0$ (no elements cannot form a positive sum)

---

### Space-Optimized Code (1D Array)
Looping backwards from `target` down to `num` ensures we only use elements from the previous iteration.

```cpp
#include <vector>

bool isSubsetSum(const vector<int>& arr, int target) {
    int n = arr.size();
    vector<bool> dp(target + 1, false);
    dp[0] = true; // base case

    for (int i = 0; i < n; i++) {
        int num = arr[i];
        for (int j = target; j >= num; j--) {
            dp[j] = dp[j] || dp[j - num];
        }
    }
    return dp[target];
}
// Time Complexity: O(n * target)
// Space Complexity: O(target)
```

---

## 2. Partition Equal Subset Sum (LC 416)

**Reduction**:
To partition an array into two subsets with equal sum, the total sum of the array $S$ must be even, and we must find a subset that sums up to $T = S/2$. 
This is mathematically identical to the Subset Sum problem with `target = totalSum / 2`.

---

## 3. Partition Array Into Two Subsets to Minimize Sum Difference

**The Problem**: Partition an array into two subsets $S_1$ and $S_2$ such that $|S_1 - S_2|$ is minimized.

### Key Insight
- Let the total sum of the array be $S$. 
- We want to minimize $|S_1 - S_2|$. Since $S_2 = S - S_1$, this is equivalent to minimizing $|S - 2 S_1|$.
- The maximum possible sum of $S_1$ is $S/2$. We want to find the largest sum $S_1 \le S/2$ that is actually possible.
- Run the Subset Sum DP table up to $S/2$. Scan the final DP table from $S/2$ downwards. The first index $j$ where `dp[j] == true` is the optimal $S_1$.
- **Minimum Difference**: $S - 2 \cdot S_1$.

```cpp
#include <vector>
#include <numeric>
#include <cmath>
#include <algorithm>

int minSubsetSumDifference(const vector<int>& arr) {
    int n = arr.size();
    int totalSum = accumulate(arr.begin(), arr.end(), 0);
    int target = totalSum / 2;

    vector<bool> dp(target + 1, false);
    dp[0] = true;

    for (int num : arr) {
        for (int j = target; j >= num; j--) {
            dp[j] = dp[j] || dp[j - num];
        }
    }

    // Find the largest j <= target that is possible
    int s1 = 0;
    for (int j = target; j >= 0; j--) {
        if (dp[j]) {
            s1 = j;
            break;
        }
    }
    return totalSum - 2 * s1;
}
// Time Complexity: O(n * totalSum)
// Space Complexity: O(totalSum)
```

---

*<- [[Knapsack_Unbounded_Knapsack|Unbounded Knapsack]] · [[Knapsack_Target_Sum|Target Sum ->]]*
