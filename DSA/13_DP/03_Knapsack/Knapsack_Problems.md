---
tags: [dsa, dp, knapsack, problems, exercises]
links: ["[[03_Knapsack_Index]]", "[[Knapsack_Target_Sum]]", "[[../04_DP_on_Strings/04_DP_on_Strings_Index]]"]
---

# Knapsack -- Problems & Exercises

*<- [[Knapsack_Target_Sum|Target Sum]] · [[../04_DP_on_Strings/04_DP_on_Strings_Index|DP on Strings ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | 0/1 Knapsack | Medium | Classic item choices | [GFG](https://www.geeksforgeeks.org/0-1-knapsack-problem-dp-10/) |
| 2 | Subset Sum Problem | Medium | Subset possibility check | [GFG](https://www.geeksforgeeks.org/subset-sum-problem-dp-25/) |
| 3 | Unbounded Knapsack | Medium | Infinite item copies forwards loop | [GFG](https://www.geeksforgeeks.org/unbounded-knapsack-repetition-items-allowed/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Partition Equal Subset Sum | Medium | Total sum partition | [LC 416](https://leetcode.com/problems/partition-equal-subset-sum/) |
| 5 | Target Sum | Medium | Sign assignment to reach target | [LC 494](https://leetcode.com/problems/target-sum/) |
| 6 | Last Stone Weight II | Medium | Minimize subset sum difference | [LC 1049](https://leetcode.com/problems/last-stone-weight-ii/) |
| 7 | Coin Change II | Medium | Unbounded combinations count | [LC 518](https://leetcode.com/problems/coin-change-ii/) |

## Tier 3 -- Multi-Dimensional Knapsack

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Ones and Zeroes | Medium | 2D capacity constraint `(m, n)` | [LC 474](https://leetcode.com/problems/ones-and-zeroes/) |
| 9 | Combination Sum IV | Medium | Permutations count (order matters) | [LC 377](https://leetcode.com/problems/combination-sum-iv/) |
| 10 | Profitable Schemes | Hard | Multi-dimensional constraint | [LC 879](https://leetcode.com/problems/profitable-schemes/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 11 | Tallest Billboard | Hard | Dual Knapsack sum balancing | [LC 956](https://leetcode.com/problems/tallest-billboard/) |
| 12 | Pizza With 3n Slices | Hard | Circular selection with gaps | [LC 1388](https://leetcode.com/problems/pizza-with-3n-slices/) |

---

## Worked Solution 1: LC 1049 -- Last Stone Weight II

**Key Insight**: You collide two stones with weights $a$ and $b$, and they leave a stone of weight $|a - b|$. We want to minimize the final remaining stone weight after colliding all stones.
- This is equivalent to dividing the stones into two groups $S_1$ and $S_2$ such that the difference between their sums $|S_1 - S_2|$ is minimized.
- This is the exact same mathematical problem as **Partition Array Into Two Subsets to Minimize Sum Difference**!

```cpp
#include <vector>
#include <numeric>
#include <algorithm>

int lastStoneWeightII(vector<int>& stones) {
    int totalSum = accumulate(stones.begin(), stones.end(), 0);
    int target = totalSum / 2;

    vector<bool> dp(target + 1, false);
    dp[0] = true;

    for (int s : stones) {
        for (int j = target; j >= s; j--) {
            dp[j] = dp[j] || dp[j - s];
        }
    }

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

## Worked Solution 2: LC 474 -- Ones and Zeroes (2D Knapsack)

**Why Tricky**: You are given an array of binary strings, and two capacities: `m` (maximum number of 0s allowed) and `n` (maximum number of 1s allowed). Find the maximum size of a subset of strings you can form.
- **State**: Standard 0/1 knapsack has 1 capacity constraint $W$. Here we have **two independent capacity constraints**: `m` and `n`.
- Let `dp[i][j]` be the maximum strings we can form with at most `i` zeros and `j` ones.
- For each string containing `zeros` count of 0s and `ones` count of 1s, we loop capacities backwards:
  `dp[i][j] = max(dp[i][j], 1 + dp[i - zeros][j - ones])`

```cpp
#include <vector>
#include <string>
#include <algorithm>

// Helper to count 0s and 1s in a string
pair<int, int> countZerosOnes(const string& s) {
    int zeros = 0, ones = 0;
    for (char c : s) {
        if (c == '0') zeros++;
        else ones++;
    }
    return {zeros, ones};
}

int findMaxForm(vector<string>& strs, int m, int n) {
    // 2D DP table initialized to 0
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    for (const string& s : strs) {
        auto [zeros, ones] = countZerosOnes(s);

        // Loop backwards for both constraints to prevent reuse
        for (int i = m; i >= zeros; i--) {
            for (int j = n; j >= ones; j--) {
                dp[i][j] = max(dp[i][j], 1 + dp[i - zeros][j - ones]);
            }
        }
    }
    return dp[m][n];
}
// Time Complexity: O(strs_count * m * n)
// Space Complexity: O(m * n)
```

---

*<- [[Knapsack_Target_Sum|Target Sum]] · [[../04_DP_on_Strings/04_DP_on_Strings_Index|DP on Strings ->]]*
