---
tags: [dsa, dp, partition-dp, problems, exercises]
links: ["[[07_Partition_DP_Index]]", "[[Partition_DP_Strange_Printer]]", "[[../08_Bitmask_DP/08_Bitmask_DP_Index]]"]
---

# Partition DP -- Problems & Exercises

*<- [[Partition_DP_Strange_Printer|Strange Printer]] · [[../08_Bitmask_DP/08_Bitmask_DP_Index|Bitmask DP ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Matrix Chain Multiplication | Medium | Classic MCM range split | [GFG](https://www.geeksforgeeks.org/matrix-chain-multiplication-dp-8/) |
| 2 | Minimum Cost Tree From Leaf Values | Medium | Leaf combination range products | [LC 95](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | Burst Balloons | Hard | Last burst range independence | [LC 312](https://leetcode.com/problems/burst-balloons/) |
| 4 | Guess Number Higher or Lower II | Medium | Minimax binary search range cost | [LC 375](https://leetcode.com/problems/guess-number-higher-or-lower-ii/) |
| 5 | Predict the Winner | Medium | Subproblem minimax subtraction | [LC 486](https://leetcode.com/problems/predict-the-winner/) |

## Tier 3 -- Game Theory & Printers

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Strange Printer | Hard | Match prefix character print reuse | [LC 664](https://leetcode.com/problems/strange-printer/) |
| 7 | Remove Boxes | Hard | 3D range state tracking trailing counts | [LC 546](https://leetcode.com/problems/remove-boxes/) |
| 8 | Minimum Score Triangulation of Polygon | Medium | Polygon chord partitions | [LC 1039](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 9 | Scramble String | Hard | 3D string partition comparison | [LC 87](https://leetcode.com/problems/scramble-string/) |
| 10 | Super Egg Drop | Hard | Minimax binary split height optimizations | [LC 887](https://leetcode.com/problems/super-egg-drop/) |

---

## Worked Solution: LC 87 -- Scramble String

**Why Tricky**: We want to check if string `s1` can be scrambled into string `s2`. We can split both strings at index `i` into two subproblems.
For each split, there are two possibilities:
1. **No swap**: We match left with left, and right with right.
   `solve(s1_left, s2_left) && solve(s1_right, s2_right)`
2. **Swap**: We match left with right, and right with left.
   `solve(s1_left, s2_right) && solve(s1_right, s2_left)`

```cpp
#include <string>
#include <unordered_map>
#include <algorithm>

using namespace std;

class Solution {
    unordered_map<string, bool> memo;

public:
    bool isScramble(string s1, string s2) {
        if (s1 == s2) return true;
        if (s1.size() != s2.size()) return false;

        string key = s1 + "#" + s2;
        if (memo.count(key)) return memo[key];

        // Anagram pruning: if characters do not match, return false immediately
        string temp1 = s1, temp2 = s2;
        sort(temp1.begin(), temp1.end());
        sort(temp2.begin(), temp2.end());
        if (temp1 != temp2) return memo[key] = false;

        int n = s1.size();
        for (int i = 1; i < n; i++) {
            // Case 1: Without Swapping
            bool noSwap = isScramble(s1.substr(0, i), s2.substr(0, i)) &&
                          isScramble(s1.substr(i), s2.substr(i));
            if (noSwap) return memo[key] = true;

            // Case 2: With Swapping (s1_left matches s2_right, s1_right matches s2_left)
            bool swapActive = isScramble(s1.substr(0, i), s2.substr(n - i)) &&
                              isScramble(s1.substr(i), s2.substr(0, n - i));
            if (swapActive) return memo[key] = true;
        }
        return memo[key] = false;
    }
};
// Time Complexity: Exponential in worst case, but anagram check and memoization reduces to polynomial in average case
// Space Complexity: O(N) stack depth + memo storage
```

---

*<- [[Partition_DP_Strange_Printer|Strange Printer]] · [[../08_Bitmask_DP/08_Bitmask_DP_Index|Bitmask DP ->]]*
