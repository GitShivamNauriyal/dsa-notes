---
tags: [dsa, dp, bitmask-dp, problems, exercises]
links: ["[[08_Bitmask_DP_Index]]", "[[Bitmask_DP_Bitmask_DP]]", "[[../09_Interval_DP/09_Interval_DP_Index]]"]
---

# Bitmask DP -- Problems & Exercises

*<- [[Bitmask_DP_Bitmask_DP|Bitmask DP Foundations]] · [[../09_Interval_DP/09_Interval_DP_Index|Interval DP ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Travelling Salesperson Problem | Medium | Visited cities bitmask | [GFG](https://www.geeksforgeeks.org/travelling-salesman-problem-set-1/) |
| 2 | Smallest Sufficient Team | Hard | Skill coverage bits matching | [LC 1125](https://leetcode.com/problems/smallest-sufficient-team/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | Can I Win | Medium | Choices bitmask minimax game | [LC 294](https://leetcode.com/problems/can-i-win/) |
| 4 | Matchsticks to Square | Medium | Partition array into 4 subsets | [LC 473](https://leetcode.com/problems/matchsticks-to-square/) |
| 5 | Beautiful Arrangement | Medium | Valid placements tracking | [LC 526](https://leetcode.com/problems/beautiful-arrangement/) |

## Tier 3 -- Advanced Matching

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Max Students Taking Exam | Hard | Broken seats row compatibility masks | [LC 1349](https://leetcode.com/problems/maximum-students-taking-exam/) |
| 7 | Number of Ways to Wear 40 Hats | Hard | Invert mapping (hats to people) mask | [LC 1434](https://leetcode.com/problems/number-of-ways-to-wear-different-hats-to-each-other/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Distribute Repeating Integers | Hard | Customer requirements subset sum | [LC 1659](https://leetcode.com/problems/maximize-grid-happiness/) |
| 9 | Find Minimum Time to Finish All Jobs | Hard | Worker allocation bitmask | [LC 1723](https://leetcode.com/problems/find-minimum-time-to-finish-all-jobs/) |

---

## Worked Solution: LC 473 -- Matchsticks to Square

**Key Insight**: We have an array of matchsticks. We want to form a square. 
- A square has 4 equal sides. The side length must be $T = \text{totalSum} / 4$.
- We can track which matchsticks have been used using a bitmask.
- Let `solve(mask, currentSideSum)` be a boolean indicating if we can form the remaining sides.
- If `currentSideSum == T`, we successfully completed one side! We reset `currentSideSum` to 0 and continue to form the remaining sides.

```cpp
#include <vector>
#include <numeric>
#include <algorithm>

using namespace std;

class Solution {
    int target;
    int memo[1 << 15]; // memo array of size 2^15 since max matchsticks is 15

    bool dfs(int mask, int currentSum, const vector<int>& matchsticks) {
        if (mask == (1 << matchsticks.size()) - 1) {
            return currentSum == 0; // successfully partitioned all matchsticks
        }

        if (memo[mask] != -1) return memo[mask];

        for (int i = 0; i < (int)matchsticks.size(); i++) {
            // If matchstick i is unused
            if (((mask >> i) & 1) == 0) {
                if (currentSum + matchsticks[i] <= target) {
                    // Calculate next sum: if it hits target, reset to 0 (side completed)
                    int nextSum = (currentSum + matchsticks[i]) == target ? 0 : currentSum + matchsticks[i];
                    if (dfs(mask | (1 << i), nextSum, matchsticks)) {
                        return memo[mask] = true;
                    }
                }
            }
        }
        return memo[mask] = false;
    }

public:
    bool makeSquare(vector<int>& matchsticks) {
        int totalSum = accumulate(matchsticks.begin(), matchsticks.end(), 0);
        if (totalSum % 4 != 0) return false;
        
        target = totalSum / 4;
        fill(begin(memo), end(memo), -1);

        // Sort in descending order to hit failures earlier (pruning)
        sort(matchsticks.rbegin(), matchsticks.rend());
        
        return dfs(0, 0, matchsticks);
    }
};
// Time Complexity: O(n * 2^n) — where n is matchsticks count (max 15)
// Space Complexity: O(2^n)
```

---

*<- [[Bitmask_DP_Bitmask_DP|Bitmask DP Foundations]] · [[../09_Interval_DP/09_Interval_DP_Index|Interval DP ->]]*
