---
tags: [dsa, dp, knapsack, target-sum]
links: ["[[03_Knapsack_Index]]", "[[Knapsack_Subset_Sum]]", "[[Knapsack_Problems]]"]
---

# Knapsack -- Target Sum & Sign Assignment

*<- [[Knapsack_Subset_Sum|Subset Sum]] · [[Knapsack_Problems|Problems ->]]*

---

## 1. Target Sum (LC 494)

**The Problem**: You are given an integer array `nums` and an integer `target`. You want to build an expression using all the integers in `nums` by assigning either `+` or `-` to each integer. Return the number of different expressions that evaluate to `target`.

---

## Mathematical Reduction to Subset Sum Count

Let's partition the array into two subsets:
1. $S_1$: subset of numbers assigned with a `+` sign.
2. $S_2$: subset of numbers assigned with a `-` sign.

From the problem statement:
$$S_1 - S_2 = \text{target}$$

We also know:
$$S_1 + S_2 = \text{totalSum}$$

Adding the two equations:
$$2 S_1 = \text{totalSum} + \text{target}$$
$$S_1 = \frac{\text{totalSum} + \text{target}}{2}$$

### Key Insights & Edge Cases:
- The problem is equivalent to finding the **number of subsets** that sum up to $S_1$.
- **Edge Case 1**: If $\text{totalSum} + \text{target}$ is odd, $S_1$ cannot be an integer. Return `0`.
- **Edge Case 2**: If $\text{totalSum} < \text{abs(target)}$, it is impossible. Return `0`.
- **Edge Case 3**: If $\text{totalSum} + \text{target} < 0$, it is impossible. Return `0`.

---

## 1D Space-Optimized Code

```cpp
#include <vector>
#include <numeric>
#include <cmath>

int findTargetSumWays(vector<int>& nums, int target) {
    int totalSum = accumulate(nums.begin(), nums.end(), 0);

    // Edge case checks
    if (totalSum < abs(target) || (totalSum + target) % 2 != 0 || (totalSum + target) < 0) {
        return 0;
    }

    int s1 = (totalSum + target) / 2;
    vector<int> dp(s1 + 1, 0);
    dp[0] = 1; // base case: 1 way to make sum 0 (empty subset)

    for (int num : nums) {
        // Loop backwards to prevent reusing the same element
        for (int j = s1; j >= num; j--) {
            dp[j] = dp[j] + dp[j - num];
        }
    }
    return dp[s1];
}
// Time Complexity: O(n * s1) where s1 = (totalSum + target) / 2
// Space Complexity: O(s1)
```

### Dry Run Variable State Table:
For `nums = [1, 1, 1, 1, 1]`, `target = 3`:
- `totalSum = 5`.
- `s1 = (5 + 3) / 2 = 4`.
- `dp` array of size 5: `[1, 0, 0, 0, 0]`.

| Num | Loop j: 4 down to 1 | `dp` Array State |
|---|---|---|
| Initial | - | `[1, 0, 0, 0, 0]` |
| 1st '1' | `dp[4]+=dp[3]=0`, ..., `dp[1]+=dp[0]=1` | `[1, 1, 0, 0, 0]` |
| 2nd '1' | `dp[4]=0`, `dp[3]=0`, `dp[2]+=dp[1]=1`, `dp[1]+=dp[0]=2` | `[1, 2, 1, 0, 0]` |
| 3rd '1' | `dp[4]=0`, `dp[3]+=dp[2]=1`, `dp[2]+=dp[1]=3`, `dp[1]+=dp[0]=3` | `[1, 3, 3, 1, 0]` |
| 4th '1' | `dp[4]+=dp[3]=1`, `dp[3]+=dp[2]=4`, `dp[2]+=dp[1]=6`, `dp[1]+=dp[0]=4` | `[1, 4, 6, 4, 1]` |
| 5th '1' | `dp[4]+=dp[3]=5`, `dp[3]+=dp[2]=10`, `dp[2]+=dp[1]=10`, `dp[1]+=dp[0]=5` | `[1, 5, 10, 10, 5]` |
- **Result**: `dp[4] = 5` (Correct: there are 5 ways).

---

*<- [[Knapsack_Subset_Sum|Subset Sum]] · [[Knapsack_Problems|Problems ->]]*
