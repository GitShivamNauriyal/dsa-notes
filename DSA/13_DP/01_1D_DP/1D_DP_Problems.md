---
tags: [dsa, dp, 1d-dp, problems, exercises]
links: ["[[01_1D_DP_Index]]", "[[1D_DP_Coin_Change]]", "[[../02_2D_Grid_DP/02_2D_Grid_DP_Index]]"]
---

# 1D DP -- Problems & Exercises

*<- [[1D_DP_Coin_Change|Coin Change]] · [[../02_2D_Grid_DP/02_2D_Grid_DP_Index|2D Grid DP ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Climbing Stairs | Easy | Fibonacci step mapping | [LC 70](https://leetcode.com/problems/climbing-stairs/) |
| 2 | Min Cost Climbing Stairs | Easy | Min of step-1/step-2 cost | [LC 746](https://leetcode.com/problems/min-cost-climbing-stairs/) |
| 3 | N-th Tribonacci Number | Easy | $F(n) = F(n-1)+F(n-2)+F(n-3)$ | [LC 1137](https://leetcode.com/problems/n-th-tribonacci-number/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | House Robber | Medium | Include/Exclude pattern | [LC 198](https://leetcode.com/problems/house-robber/) |
| 5 | House Robber II | Medium | Circular house constraint | [LC 213](https://leetcode.com/problems/house-robber-ii/) |
| 6 | Coin Change | Medium | Unbounded choices minimum | [LC 322](https://leetcode.com/problems/coin-change/) |
| 7 | Perfect Squares | Medium | Min perfect squares summing to n | [LC 279](https://leetcode.com/problems/perfect-squares/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Decode Ways | Medium | 1-digit vs 2-digits decoding | [LC 91](https://leetcode.com/problems/decode-ways/) |
| 9 | Word Break | Medium | Split check via substring prefixes | [LC 139](https://leetcode.com/problems/word-break/) |
| 10 | Partition Equal Subset Sum | Medium | 1D boolean array knapsack | [LC 416](https://leetcode.com/problems/partition-equal-subset-sum/) |
| 11 | Integer Break | Medium | Optimal product decomposition | [LC 343](https://leetcode.com/problems/integer-break/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 12 | Counting Bits | Easy/Medium | Bitwise recurrence mapping | [LC 338](https://leetcode.com/problems/counting-bits/) |
| 13 | Word Break II | Hard | DP backtracking path reconstruction | [LC 140](https://leetcode.com/problems/word-break-ii/) |
| 14 | Frog Jump | Hard | 2D-state representation `(position, last_jump)` | [LC 403](https://leetcode.com/problems/frog-jump/) |

---

## Worked Solution 1: LC 213 -- House Robber II (Circular Street)

**Key Insight**: The houses are arranged in a circle. This means the first house is adjacent to the last house; you cannot rob both. 
- **Solution**: Split the problem into two separate linear House Robber subproblems:
  1. Rob houses from index `0` to `n-2` (exclude the last house).
  2. Rob houses from index `1` to `n-1` (exclude the first house).
- The answer is the maximum of these two results.

```cpp
#include <vector>
#include <algorithm>

// Linear House Robber solver (Space Optimized)
int robLinear(const vector<int>& nums, int start, int end) {
    if (start == end) return nums[start];
    
    int prev2 = nums[start];
    int prev1 = max(nums[start], nums[start + 1]);
    int curr = prev1;

    for (int i = start + 2; i <= end; i++) {
        curr = max(prev1, nums[i] + prev2);
        prev2 = prev1;
        prev1 = curr;
    }
    return curr;
}

int rob(vector<int>& nums) {
    int n = nums.size();
    if (n == 1) return nums[0];
    if (n == 2) return max(nums[0], nums[1]);

    // Max of: rob 0 to n-2 OR rob 1 to n-1
    return max(robLinear(nums, 0, n - 2), robLinear(nums, 1, n - 1));
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

## Worked Solution 2: LC 416 -- Partition Equal Subset Sum

**Key Insight**: The problem asks if we can partition an array into two subsets with equal sum. 
- Let the total sum of the array be $S$. If $S$ is odd, we cannot partition it equally (return `false`).
- If $S$ is even, we want to find if there is a subset that sums to $T = S/2$. This is a boolean 0/1 knapsack variant.
- Let `dp[j]` be a boolean indicating if a subset sum of `j` is possible.
- **Recurrence**: For each number `num` in `nums`, update `dp[j] = dp[j] || dp[j - num]` backwards from $T$ to `num` (backwards prevents reusing the same element multiple times).

```cpp
#include <vector>
#include <numeric>

bool canPartition(vector<int>& nums) {
    int totalSum = accumulate(nums.begin(), nums.end(), 0);
    if (totalSum % 2 != 0) return false; // odd sum cannot be divided equally

    int target = totalSum / 2;
    vector<bool> dp(target + 1, false);
    dp[0] = true; // base case: sum 0 is always possible (empty subset)

    for (int num : nums) {
        // Iterate backwards from target down to num
        for (int j = target; j >= num; j--) {
            dp[j] = dp[j] || dp[j - num];
        }
    }
    return dp[target];
}
// Time Complexity: O(n * target) — where n is array size, target is sum/2
// Space Complexity: O(target)
```

### Dry Run Variable State Table:
For `nums = [1, 5, 11, 5]`. Sum = 32, Target = 16.
- `dp` array of size 17 initialized to `false`, `dp[0] = true`.

| Num | Update Path (j) | `dp[j]` values updated to `true` |
|---|---|---|
| 1 | 16 down to 1 | `dp[1] = true` |
| 5 | 16 down to 5 | `dp[6] = true` (from `dp[1]`), `dp[5] = true` (from `dp[0]`) |
| 11 | 16 down to 11 | `dp[17] (N/A)`, `dp[16] = true` (from `dp[5]`), `dp[12] = true` (from `dp[1]`), `dp[11] = true` (from `dp[0]`) |
- At index 16, `dp[16]` became `true`. We can stop early or continue.
- **Result**: `true`.

---

*<- [[1D_DP_Coin_Change|Coin Change]] · [[../02_2D_Grid_DP/02_2D_Grid_DP_Index|2D Grid DP ->]]*
